<div align="center">

![Python 3.10+](https://img.shields.io/badge/python-3.10.13-green.svg)
![Selenium](https://img.shields.io/badge/Selenium-4.15.2-red)

</div>





## Introduction

This repo follows the data and implementation of the paper [WebVoyager](https://arxiv.org/abs/2401.13919). WebVoyager is an innovative Large Multimodal Model (LMM) powered web agent that can complete user instructions end-to-end by interacting with real-world websites. 

And now, we focus on catch lyrics problem.

- **Multimodal Web Agent**. We implement WebVoyager that integrates textual and visual information to address web tasks end-to-end and introduce a generalist planning approach for navigation.
- **Online Environment**. We build an online web browsing environment using Selenium. 
- **Diverse Web Tasks** We offer a variety of tasks centered on widely used websites and introduce a
method for expanding these tasks.
- **Evaluation Tool** We propose an automated evaluation protocol using GPT-4V.

## Setup Environment

We use Selenium to build the online web browsing environment. 
 - Make sure you have installed Chrome. (Using the latest version of Selenium, there is no need to install ChromeDriver.)
 - If you choose to run your code on a Linux server, we recommend installing chromium. (eg, for CentOS: ```yum install chromium-browser```) 
 - Create a conda environment for WebVoyager and install the dependencies.
    ```bash
    conda create -n webvoyager python=3.10
    conda activate webvoyager
    pip install -r requirements.txt
    ```

## Data

### Overview
The test data you can found in `data/WebVoyager_datatasks_test.jsonl`.

And now, you can modify to your own.
```
{"web_name": "Genius", "id": "Genius--1", "ques": "Search for the song 'Let It Go'. Identify the result with the highest number of views, open the lyrics page, scroll down if necessary, and extract the complete lyrics.", "web": "https://genius.com/"}
```


Other detail, please follow the [WebVoyager github](https://github.com/MinorJerry/WebVoyager)
## Running

### Running WebVoyager
After setting up the environment, you can start running WebVoyager. 

 1. Copy the examples you want to test into `data/tasks_test.jsonl`. For Booking and Google Flights tasks, please manually update the date in the task if it is outdated.
 2. Modify the api_key in `run.sh` 

You can run WebVoyager with the following command:
```bash 
bash run.sh
```

The details of `run.sh`:
```bash 
#!/bin/bash
nohup python -u run.py \
    --test_file ./data/tasks_test.jsonl \
    --api_key YOUR_OPENAI_API_KEY \
    --headless \
    --max_iter 15 \
    --max_attached_imgs 3 \
    --temperature 1 \
    --fix_box_color \
    --seed 42 > test_tasks.log &
```

For WebVoyager (Text only), an example script can be:
```bash 
#!/bin/bash
nohup python -u run.py \
    --test_file ./data/tasks_test.jsonl \
    --api_key YOUR_OPENAI_API_KEY \
    --headless \
    --max_iter 15 \
    --max_attached_imgs 1 \
    --temperature 1 \
    --text_only \
    --api_model gpt-4-1106-preview \
    --seed 42 > test_tasks_text_only.log &
```


### Parameters

General:
- `--test_file`: The task file to be evaluated. Please refer to the format of the data file in the `data`.
- `--max_iter`: The maximum number of online interactions for each task. Exceeding max_iter without completing the task means failure.
- `--api_key`: Your OpenAI API key.
- `--output_dir`: We should save the trajectory of the web browsing.
- `--download_dir`: Sometimes Agent downloads PDF files for analysis.

Model:
- `--api_model`: The agent that receives observations and makes decisions. In our experiments, we use `gpt-4-vision-preview`. For text-only setting, models without vision input can be used, such as `gpt-4-1106-preview`.
- `seed`: This feature is in Beta according to the OpenAI [Document](https://platform.openai.com/docs/api-reference/chat). 
- `--temperature`: To control the diversity of the model, note that setting it to 0 here does not guarantee consistent results over multiple runs.
- `--max_attached_imgs`: We perform context clipping to remove outdated web page information and only keep the most recent k screenshots.
- `--text_only`: Text only setting, observation will be accessibility tree.

Web navigation:
- `--headless`: The headless model does not explicitly open the browser, which makes it easier to deploy on Linux servers and more resource-efficient. Notice: headless will affect the **size of the saved screenshot**, because in non-headless mode, there will be an address bar.
- `--save_accessibility_tree`: Whether you need to save the Accessibility Tree for the current page. We mainly refer to [WebArena](https://github.com/web-arena-x/webarena) to build the Accessibility Tree.
- `--force_device_scale`: Set device scale factor to 1. If we need accessibility tree, we should use this parameter.
- `--window_width`: Width, default is 1024.
- `--window_height`: Height, default is 768. (1024 * 768 image is equal to 765 tokens according to [OpenAI pricing](https://openai.com/pricing).)
- `--fix_box_color`: We utilize [GPT-4-ACT](https://github.com/ddupont808/GPT-4V-Act), a Javascript tool to extracts the interactive elements based on web element types and then overlays bounding boxes. This option fixes the color of the boxes to black. Otherwise it is random.

### Develop Your Prompt

Prompt optimisation is a complex project which directly affects the performance of the Agent. You can find the system prompt we designed in `prompts.py`. 

The prompt we provide has been tweaked many times, but it is not perfect, and if you are interested, you can **do your own optimisation** without compromising its generality (i.e. not giving specific instructions for specific sites in the prompt). If you just need the Agent for some specific websites, then giving specific instructions is fine.

If you want to add Action to the prompt, or change the Action format, it is relatively easy to change the code. You can design your own `extract_information` function to parse the model's output and modify `run.py` to add or modify the way of action execution.

## Results and Evaluation

The results will be saved in the output dir you set, in this experiment we use `results` and we put some of the results in `results/examples`.  The results folder for each task contains interact messages and several screenshots.

</div>

