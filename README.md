# BMAD-expansion-example

This is an example of an expansion pack for the BMAD-METHOD, an agentic coding framework, that can be used for multimodal OSINT/FIMI analysis. This expansion pack was created by and for Claude Code. Similar expansion packs can be made by prompting Claude Code with your desired tasks and with this expansion pack's structure as a guide.

## Installation

Ensure the ```expansion-packs``` directory is in the root directory of your project. Then run:

```bash
cd /path/to/your-project
npx bmad-method@4.43.0 install
```

Follow the installation prompts (if needed consult [this](https://youtu.be/LorEJPrALcg?si=kNbLm6-fKJIJ52WL) tutorial).

This will automatically detect and install the ```multimodal-osint``` expansion pack.

## Use

Pull up Claude Code, and use the command ```/multimodal-osint\:agents:multimodal-investigator```. This will initiate a session with the ```multimodal-investigator``` agent, an expert in multimodal OSINT.

## Structure

```
multimodal-osint
├── agents
│   └── multimodal-investigator.md
├── config.yaml
├── data
│   ├── disarm-framework-reference.md
│   ├── manipulation-playbook.md
│   └── platform-behavior-baselines.md
├── README.md
├── tasks
│   ├── execute-hunt.md
│   ├── init-data.md
│   ├── logos.md
│   ├── suggest-technique.md
│   └── vision.md
├── templates
├── workflows
│   └── disarm-hunt.yaml
```

The ```agents``` directory contains markdown files specifying the agents' behavior. The ```data``` folder contains markdowns with reference information for the agent (a lot of it is in Traditional Chinese due to where I got it from), and the ```tasks``` folder contains specifications for different tasks the agent is able to execute. ```logos.md``` and ```vision.md``` use the ```logos``` directory and ```vision.ipynb``` notebook provided in this repo, respectively.

```config.yaml``` configurates the expansion pack (though here it's only one agent), and ```templates``` is currently empty but can store example templates for output, etc.

The main task the agent carries out, ```hunt```, takes in a multimodal dataset (typically social media data) and aims to use vision tools, logo identification, and text analysis to detect DISARM TTPs (disinformation techniques). It outputs its findings into a markdown file.

## Make one yourself!

As mentioned above, I created this expansion pack (based off of a different OSINT pack) by prompting Claude Code with my desired tasks and by asking it to use the other pack as a reference. Here is the prompt I used:

```
I want to make a multimodal FIMI/DISARM pack for BMAD in expansion-packs/multimodal-osint. You can reference expansion-packs/osint-core and expansion-packs/v1 for structure. 

This pack will have one agent, to be described in multimodal-investigator.md. The agent should have a bubbly personality but be detail-oriented, systematic, and politically unbiased.

The agent’s main task is to assess the data it is given for DISARM TTPs (methods for disinformation). Reference expansion-packs/multimodal-osint/data for details. 

The input data will take the form of social media posts, with exact structure unknown. The data will be multimodal. THE AGENT WILL WORK WITH THE DATA IN ITS NATIVE FORMAT, ASSESSING ITS STRUCTURE ON THE FLY. With osint-core as a guide, detail this process in multimodal-osint/tasks/execute-hunt.md. The agent will have access to the following subtasks:

The agent will have access to a file vision.ipynb which it should make a copy of and use for VLM capabilities. Detail this task in multimodal-osint/tasks/vision.md.

The agent also has access to a folder called logos, which contains a large number of logo images and a csv explaining them. By using this folder in tandem with vision, the agent can identify specific logos in images. Detail this task in multimodal-osint/tasks/logos.md.

The agent will explore MULTIPLE DISARM TTPs as it sees fit, using both exploration and exploitation, and write a report similar to the one in experiment/FIMI_Investigation_Report.md. TAILOR THE MAIN TASK, which will be described in execute-hunt.md, AS NEEDED TO PRODUCE SUCH A REPORT.

Complete ALL config and yaml files and add files in multimodal-osint as needed to make it a complete expansion pack.
```

(I specified the agent's personality so I would be able to gauge if the model was slipping "out of character", given that this BMAD framework is all one big roleplay prompt.)

It *is* possible to write the expansion pack by hand, but given the number of things that it is necessary to include, Claude may or may not do a better and faster (though more expensive) job. I would recommend automating the process with Claude Code, unless you are an expert in how you want the task to be carried out.
