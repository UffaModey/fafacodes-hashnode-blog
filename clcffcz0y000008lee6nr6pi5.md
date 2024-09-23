---
title: "Open AI API Automation with Linux Cron Job"
datePublished: Mon Jan 02 2023 23:20:26 GMT+0000 (Coordinated Universal Time)
cuid: clcffcz0y000008lee6nr6pi5
slug: open-ai-api-automation-with-linux-cron-job
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1672694678610/69161cd8-a2f2-4ea0-8af0-0f8089bd120b.jpeg
tags: linux, automation, cronjob, openai, text-prediction

---

# Objectives 🚀

* Introduction to [Cron](https://crontab.guru/) for automating Linux tasks.
    
* Use [Open AI API](https://beta.openai.com/docs/api-reference/completions/create) for text completion predictions.
    

# Prerequisites 🎢

1. Linux OS on the user's system.
    
2. Root or sudo privileges for the system user.
    
3. Basic ability to use the command line interface.
    
4. Cron is often already pre-installed in your Linux system. Install cron using `sudo apt install cron` if it is not installed.
    
5. A user account on [Open AI with access to API secret keys.](https://beta.openai.com/account/api-keys)
    

# What is a cron job? 🚧

[Cron](https://en.wikipedia.org/wiki/Cron) is a popular Linux tool used for workload automation. Using Cron, a user can schedule Linux tasks that will run automatically after the specific date/time or interval defined during the job set-up.

Cron jobs are created as shell scripts or commands defined in a crontab file. The cron daemon runs continually as a background service to monitor the crontab files and execute them based on the preset schedule. Every minute, Cron examines all stored crontabs and sees if any command needs to be executed in the current minute.

# Setting up a Cron Job ⛔

## 1\. Create a Task for Automated Execution

Pull the task code from the [remote repo in GitHub](https://github.com/UffaModey/open_ai_cron_job). Follow the steps in the `README.md` file to run the job defined in the `main.py` file.

A Python script is used to query the Open AI API for text completion prediction. A request to the API generates three marketing tagline messages that are related to a specified word and updates the response as a new line in a text file `output.txt`.

```python
import openai
from datetime import datetime
from dotenv import load_dotenv
load_dotenv()

import os

openai.api_key = os.environ.get('OPENAI_API_KEY')

def generate_message():
    word = "love"
    response = openai.Completion.create(
        model="text-davinci-002",
        prompt=generate_prompt(word),
        temperature=0.6,
    )
    return response

def generate_prompt(word):
    return """Suggest three word and a marketing message.

Word: happy
Message: Act the way you want to feel, Do not Be Afraid To Be Happy, Happiness Goes Where Happiness Is
Word: tall
Message: Practice safety at a height, The height of quality, We go to great heights for you
Word: {}
Message:""".format(
        word.capitalize()
    )

result = generate_message()
text = result.choices[0]['text']

def write_to_file(filename, output):
    with open(filename, 'a') as file:
        time = str(datetime.now())
        file.write(time + ' ' + output + '\n')

write_to_file('output.txt', text)
```

## 2\. Check for **Active Cron Jobs Running**

List all cron jobs that have been scheduled by the current user.

```bash
 crontab -l
```

## 3\. Open the crontab File for Editing

Open a configuration file called a crontab (cron table). This opens using the default editor for your system. If you are opening the crontab for the first time, you may be asked to select your default editor.

```bash
 crontab -e
```

## 4\. Add a Cron job in the crontab

Include the line below to the opened crontab file. This job changes to the project directory, activates a virtual environment and runs the `main.py` file every minute.

```plaintext
* * * * * cd /Users/uffamodey/projects/open_ai_cron_job ; source bin/activate ; python3 main.py
```

Save and exit the editor.

Use `crontab -l` to confirm that the cron job is running.

# Task Review 🏆

The crontab file specifies the intervals at which Cron should run the commands. The commands are separated using semi-colons.

The first five arguments represent the execution time. The sixth argument is the command to be executed.

```plaintext
[minute] [hour] [day of month] [month] [day of week] [command]
```

| **Field** | **Allowed values** |
| --- | --- |
| minute | 0-59 |
| hour | 0-23 |
| day of month | 1-31 |
| month | 1-12 (or names: JAN - DEC) |
| day of week | 0-6 (or names: SUN - SAT) |

| **Special field value** | **Description** | **Example** |
| --- | --- | --- |
| Asterisk | An asterisk represents every allowed value (first to last). | \* (run every hour, month, etc.) |

# Conclusion ✅

This tutorial showcase how a Cron job may be set up on a Linux machine to automatically run a Python script every minute. The Python script contains a function that generates catchy marketing taglines that relates to a specified word using Open AI text completion prediction API. The output of the function is updated as a new line of a text file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672700882502/5ac23b97-b9f7-446a-ae33-197c660e047c.png align="center")

# Further Reading 📙

* [How to Use Cron to Automate Linux Jobs on Ubuntu 20.04](https://www.cherryservers.com/blog/how-to-use-cron-to-automate-linux-jobs-on-ubuntu-20-04#:~:text=in%20Linux%20community.-,What%20is%20Cron%3F,on%20a%20pre%2Dset%20schedule.)
    
* [How to List, Display, & View all Current Cron Jobs in Linux](https://phoenixnap.com/kb/how-to-list-display-view-all-cron-jobs-linux#:~:text=Listing%20Cron%20Jobs%20in%20Linux,-How%20to%20List&text=You%20can%20find%20them%20in,crontab%20for%20the%20whole%20system.&text=In%20RedHat%2Dbased%20systems%2C%20this,located%20at%20%2Fetc%2Fcron.)
    
* [Open AI Quickstart tutorial](https://beta.openai.com/docs/quickstart)