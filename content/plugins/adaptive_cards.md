---
date: 2000-01-01T00:00:00+00:00
title: Plugin Cards
author: eoinmcafee
weight: 3
---

# Drone Announces User Interface Plugins, Powered by Microsoft Adaptive Cards

A powerful new feature that allows Drone Plugins to extend the Build Execution view and provide pertinent information of their execution.

Plugin Cards allow Drone users to get high level insights into their build’s execution, and enables them to detect potential issues faster.

Plugin Cards will automatically appear in the build’s execution view, at the end of the plugin execution, if a supported plugins is used.

## What does a card look like

Different plugins will have different visualizations depending on the type of data presented.
Here is an example of what a card for the Docker plugin looks like :

![Card](/screenshots/card.png)

This card allows you to see, at a glance, all of the important information pertaining to the image generated during the execution of the docker plugin. 

## Supported Versions
    - Drone Server - v2.6.0+
    - Drone Runner Docker - v1.8.0+ / Drone Kubernetes Runner - latest
## Supported plugins:
    - Drone-docker
    - Snyk
    - Gitleaks 

# How do I add adaptive cards support to my plugin?  

Plugin Cards are built upon the open adaptive cards format . This makes it much easier for a plugin developer to add enhanced output functionality without requiring any changes to the Drone Server.

A card is made up of a template (what the user sees) and output from a plugin (raw data from the build). These are combined by Drone to display the card in the Drone UI.

During the plugin’s execution, Drone downloads the plugin template. An example of what a template looks like can be found here and it is downloaded when rendering the card. 

Below is the JSON payload that is generated by the docker plugin. This is the raw information that appears on the card:

{{< highlight yaml "linenos=table" >}}
{
	"Id": "sha256:d9437eae98f2787931b0fae86cc79c3993cd3ed031f84f67a41fdff5bcb436c9",
	"RepoTags": ["drone-test:latest", "5699c94024237b8b4838717f4f7a55d55cab66bb:latest"],
	"RepoDigests": ["drone-test@sha256:2ab4cd8ec41a0f7d68c34289a52dffa4d483abbdd10bcd68fd4da3e904a81f76"],
	"Parent": "sha256:bdca5d527305e375f37d177e745224d0d7fce56eb1454210d9a6f2441a40d31b",
	"Comment": "",
	"Created": "2022-01-05T12:04:34.3483105Z",
	"Container": "eafafc780d73493be11f546777ff303341b2094671b244ed804eff49a722d40a",
	"DockerVersion": "20.10.9",
	"Author": "",
	"Architecture": "amd64",
	"Os": "linux",
	"Size": 186298129,
	"VirtualSize": 186298129,
	"Metadata": {
		"LastTagTime": "2022-01-05T12:04:34.392555Z"
	}
}
{{< / highlight >}}

# Adding Drone card functionality to a plugin

## Creating a template
To get started you first want to think about the information you wish to display on-screen & how it should look. This becomes the card template. The easiest way to create it is by using this adaptive cards designer.

The docker template in this example is hosted and served using Github pages.
Mention the directory structure of the plugin
    - /docs/template.json
    - /docs/sample_data.json

## Card Input
This assumes that your plugin has been written in Golang but it could be written in any other language. As long as the JSON output is created correctly.

The structure of a card struct is as follows. This is what is written to the logs for the runner to extract and publish information to the Drone server.

{{< highlight golang "linenos=table" >}}
CardInput struct {
    Schema string          `json:"schema"` // template url
    Data   json.RawMessage `json:"data"` // template data
}
{{< / highlight >}}

The following encodes the card data in order for the runner to extract it from the logs:
{{< highlight go "linenos=table" >}}
func writeCardTo(out io.Writer, data []byte) {
    encoded := base64.StdEncoding.EncodeToString(data)
    io.WriteString(out, "\u001B]1338;")
    io.WriteString(out, encoded)
    io.WriteString(out, "\u001B]0m")
    io.WriteString(out, "\n")
    }
}
{{< / highlight >}}

This code allows the plugin to support both CI & Drone. Once the card input struct has been created, it must be written to “dev/stdout”. 

{{< highlight go "linenos=table" >}}
func writeCard(path string, card interface{}) {
    data, _ := json.Marshal(card)
    switch {
    case path == "/dev/stdout":
        writeCardTo(os.Stdout, data)
    case path == "/dev/stderr":
        writeCardTo(os.Stderr, data)
    case path != "":
        ioutil.WriteFile(path, data, 0644)
    }
}
{{< / highlight >}}

Once the input data is written, the runner & Drone server will do the rest and in the build screen, a card should appear. 