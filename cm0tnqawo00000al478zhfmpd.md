---
title: "Pictures2Stories: Transforming Images into Short Stories with AI and Azure Services"
datePublished: Sun Sep 08 2024 14:16:27 GMT+0000 (Coordinated Universal Time)
cuid: cm0tnqawo00000al478zhfmpd
slug: pictures2stories-transforming-images-into-short-stories-with-ai-and-azure-services
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724706807404/9d8877ad-5031-47a5-a022-95627c0aba26.jpeg
tags: ai, postgresql, azure, django, openai, python-projects, azure-app-services, chatgpt, image-analysis

---

# Overview

Pictures2Stories is a Django-based web application that uses [Azure AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview), [OpenAI Chat Completion API](https://platform.openai.com/docs/guides/chat-completions/getting-started), and [Azure’s App Service and Database for PostgreSQL](https://azure.microsoft.com/en-us/products/postgresql) to create short, adventure-themed stories based on three images that the user uploads.

The process begins with Azure AI Vision analyzing each uploaded image to generate a high-confidence caption that accurately describes the image's content. These descriptive captions serve as prompts for the OpenAI GPT-3.5 model, which, acting as a children's book author from the 1980s, crafts a captivating adventure and fiction story. The generated story and the reference images are stored in the app’s PostgreSQL database, making it easily accessible and viewable on the homepage.

The application's source code is hosted on GitHub, and [Azure App Service](https://azure.microsoft.com/en-us/products/app-service) is used for deployment. Continuous integration and deployment (CI/CD) pipelines are set up to automatically push updates to the app. The app’s data is securely managed using Azure Database for PostgreSQL, which is seamlessly integrated when the app is created on Azure.

This blog post provides a detailed, step-by-step guide to developing the application, including how to set up and configure the Azure services used. Additionally, it offers insights into potential areas for further development and provides resources for learning more about the Azure technologies leveraged in this project.

# Demo

%[https://youtu.be/b1Ia6M6aqaM] 

# Prerequisites

* **An Azure account with an active subscription.** If you don't have an Azure account, you can [create one for free](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?icid=python).
    
* **Basic knowledge of Python and Django.** Familiarity with these technologies is essential for following this tutorial.
    
* **A GitHub account**. A good understanding of git and git commands is required.
    
* **Python 3.7 or higher.** Ensure that you are using the correct Python version by running `python3 --version`. This tutorial requires Python 3.7 or later.
    
* **A Linux-based PC.** This tutorial is conducted using Linux commands.
    

# Set Up the Project on GitHub

* You can fork the project from its GitHub repo link below or follow the steps below to create a new project from scratch.
    

[https://github.com/UffaModey/pictures2stories](https://github.com/UffaModey/pictures2stories)

* Create a new GitHub repository for the project.
    
    * set a unique **Repository name** for the project and fill in a **Description.**
        
    * Set the project to **Public**.
        
    * Add a **README.md** file and a **.gitignore** for Python.
        
    * **Choose a licence** for the project.
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723313755034/e54b17e8-6516-4348-814e-9c51a77f1eb6.png align="center")

* Clone the project to your local computer terminal using `git clone` [`https://github.com/UffaModey/pictures2stories.git`](https://github.com/UffaModey/pictures2stories.git)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723314402334/e7d1bf52-a670-4e8a-a6a5-66c21b72a0f9.png align="center")

# Creating a Django App in PyCharm

## Setting up a Django Project

* Open the project folder in PyCharm.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723314901648/cdd5f083-7063-4b14-92b1-8b61398c8134.png align="center")

* Create and activate a virtual environment for the project in the Pycharm terminal using the command below.
    

```bash
python3 -m venv .venv
source .venv/bin/activate
```

* Create a `requirements.txt` file in the project folder and update it with the following packages to be installed in the virtual environment.
    

```bash
asgiref==3.8.1
Django==4.2.7
django-redis==5.3.0
djangorestframework==3.13.1
pillow==10.4.0
psycopg2-binary==2.9.9
python-dotenv==1.0.0
pytz==2024.1
redis==5.0.8
sqlparse==0.5.1
whitenoise==6.6.0
azure-ai-vision-imageanalysis==1.0.0b2
azure-core==1.30.1
openai==1.14.1
```

* Run the command below in your terminal with the virtual environment activated to install the dependencies in the virtual environment.
    

```bash
pip install -r requirements.txt
```

* Start a Django project using the command below in the project base directory. This process also creates some new files in your project directory.
    
    * **init.py** is an empty file that functions to tell Python that this directory should be considered a package.
        
    * **settings.py** contains all of your settings or configurations.
        
    * **urls.py** contains the URLs within the project.
        
    * **asgi.py** and **wsgi.py** serve as the entry points for your web servers depending on what type of server is deployed.
        

```bash
django-admin startproject pictures2stories .
```

* Run the following command to start the server locally to ensure that is working correctly.
    

```bash
python manage.py runserver
```

* Create an app for the project using the following command.
    

```bash
python manage.py startapp azureai
```

* Register the app with the project by updating the **INSTALLED\_APPS** variable inside **settings.py** for the project. Include the line to the `INSTALLED_APPS` list to tell Django that this app needs to be included when it runs the project. Your updated `INSTALLED_APPS` list should look like the following
    

```bash

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'azureai.apps.AzureaiConfig',
]
```

* Create a view and register the path inside a `URLconf`. Open **views.py** file in **azureai** directory and update it with the following.
    

```python
from django.shortcuts import render
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world!")
```

* Create a file named **urls.py** in the **azureai** directory and update it with the following.
    

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

* Register the newly created `URLconf` in the **azureai** application in the core **urls.py** file which is inside the **pictures2stories** directory.
    

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('', include('azureai.urls')),
    path('admin/', admin.site.urls),
]
```

* Start the server again using `python manage.py runserver` and view **Hello, world!** in your browser window when you open [http://localhost:8000/](http://localhost:8000/) in your preferred browser.
    

## Models and Working with Application Data

Models enable us to define essential data fields and the behaviour of the data. Let us add the data models that will be required for the application.

* Open *azureai/models.py* in PyCharm.
    
* Add two classes which we will use to contain the models.
    

```python
import uuid
from django.db import models


class Story(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(max_length=255, default="New Story")
    generated_story = models.TextField()

    def __str__(self):
        return self.title

    class Meta:
        verbose_name_plural = "Stories"

class Image(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    image = models.ImageField(upload_to='images/')
    story = models.ForeignKey(Story, related_name='stories',
                                on_delete=models.CASCADE, null=True, blank=True)

    caption = models.TextField()  # Stores the caption generated by Azure Vision API for the image
    caption_confidence = models.FloatField()

    def __str__(self):
        return self.caption
```

* Create migrations for azureai app `python manage.py makemigrations azureai`
    
* Update the database `python` [`manage.py`](http://manage.py) `migrate`
    
* View the data models using Django admin site. Create a super user using the following command .
    

```python
python manage.py createsuperuser
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723322255440/494c2307-349a-4e9c-8914-18c6c4cb54db.png align="center")

* Log in to the Django admin site using the admin site URL [http://](http://127.0.0.1:8000/admin/login/?next=/admin/)[127.0.0.1](http://localhost)[:8000/admin](http://127.0.0.1:8000/admin/login/?next=/admin/)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723322420015/caf28a67-ce62-4a14-a42d-62bb6d9cc096.png align="center")

* Register your models in **azureai/admin.py** to enable the admin site to access your data.
    

```python
# Register your models here.
from django.contrib import admin
from .models import Image, Story

admin.site.register(Image)
admin.site.register(Story)
```

* Now in the Django admin site, you should be able to view the models for *Images* and *Stories* and create objects for them.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723323330584/37cf05aa-ec66-4e46-aee7-17d52d168c9e.png align="center")

## Creating Backend Views and Frontend Templates

A view is responsible for displaying the required information to a user. Each view in Django could load or modify data and return an HTML template with the information to the user.

* Register a path for the first view. To begin, register a path for the first view we'll create. This path will map the application's URL to a specific function call. For instance, a request to `www.`[`localhost:8000/stories`](http://localhost:8000/stories) should trigger a function that fetches all the stories from the application database, allowing the user to view them. Similarly, a request to `www.`[`localhost:8000/stories/<story_id>`](http://localhost:8000/stories/%3Cstory_id%3E) should trigger a function that retrieves the specific story from the database based on the `story_id` included in the URL.
    
* Open **azureai/views.py**
    

```python
from django.shortcuts import render, get_object_or_404, reverse
from django.http import HttpResponseRedirect
from django.urls import reverse_lazy
from django.views.generic.edit import DeleteView
from django.views.decorators.cache import cache_page
from django.views.decorators.csrf import csrf_exempt

from azureai.models import Image, Story


class StoryDeleteView(DeleteView):
    model = Story
    template_name = 'pictures_2_stories/story_confirm_delete.html'
    success_url = reverse_lazy('story_delete_success')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['story'] = self.object
        return context

def story_delete_success(request):
    print('Request for story delete success received')
    return render(request, 'pictures_2_stories/delete_story_success.html')

def stories_list(request):
    print('Request for index page received')
    stories = Story.objects.all()
    return render(request, 'pictures_2_stories/index.html', {'stories': stories})

@cache_page(60)
def story_details(request, id):
    print('Request for stories details page received')
    story = get_object_or_404(Story, id=id)
    images = Image.objects.filter(story=story)
    return render(request, 'pictures_2_stories/details.html', {'story': story, 'images': images})

def create_story(request):
    print('Request for add story page received')
    return render(request, 'pictures_2_stories/create_story.html')

@csrf_exempt
def add_story(request):
    if request.method == 'POST':
        try:
            # Fetch the uploaded images
            image_one = request.FILES.get('image_one')
            image_two = request.FILES.get('image_two')
            image_three = request.FILES.get('image_three')
        except KeyError as e:
            # Redisplay the form with an error message if any image is missing
            return render(request, 'pictures_2_stories/create_story.html', {
                'error_message': e,
            })
        
        try:
            # Save the story and images to the database
            story = Story(generated_story="new story content",
                          title="new story")
            story.save()
        except Exception as e:
            return render(request, 'pictures_2_stories/create_story.html', {
                'error_message': e,
            })

        return HttpResponseRedirect(reverse('story_details', args=(story.id,)))

    else:
        return render(request, 'pictures_2_stories/add_story.html')
```

* To make the views we have created callable, we need to create the URLconf in **azureai/urls.py**. Update this file using the following code.
    

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.stories_list, name='stories_list'),
    path('create', views.create_story, name='create_story'),
    path('add', views.add_story, name='add_story'),
    path('story/<uuid:id>', views.story_details, name='story_details'),
    path('story/<uuid:pk>/delete', views.StoryDeleteView.as_view(), name='story_delete'),
    path('story-delete-success', views.story_delete_success, name='story_delete_success'),
    # More patterns to come later
]
```

* Update **settings.py** file
    

```python
STATIC_URL = 'static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]

MEDIA_URL = 'media/'

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

* Update **pictures2stories/urls.py** with the following line of code
    

```python
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

In **azureai** directory, create a folder called **templates** and inside **templates,** create another folder called **pictures\_2\_stories.** In this folder create and update the following HTML files. Vice the code for all of these files in the project repo [here](https://github.com/UffaModey/pictures2stories/tree/main/azureai/templates/pictures_2_stories).

* **base.html** - A base template for all the views. It contains the CSS styling information and the app navigation bar.
    
* **index.html** - The homepage view for the app. All of the stories in the application database are displayed here. A user can also create a new story from this view.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723494074542/780193dd-5a64-4f1c-89f8-ac7635fc30a8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723494482219/ec8c3564-646c-4502-9d15-d5a37dfc23f8.png align="center")

* **create\_story.html** - This view enables a user to create a new story by uploading three images.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723494165357/444e6f2c-dd8b-48fc-9cc4-bc84dc69bfc2.png align="center")

* **details.html** - A user can view a story's full content, including the pictures related to the story.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723494223644/e20a30b0-741c-4935-8a7c-77790139e7b2.png align="center")

* **story\_confirm\_delete.html** \- Displays a confirmation prompt when a user attempts to delete a story, with options to confirm or cancel the deletion.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724711658269/7447e1a8-d3c9-489a-8b29-3680011aab0b.png align="center")

* **delete\_story\_success.html** - Shows a confirmation message after a story is successfully deleted, with a link to return to the homepage.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724711691244/1fc5d33a-f1ce-4353-a78e-03605596165a.png align="center")

# AzureAI Services: Azure AI Vision and OpenAI Chat Completion

This section of the application essentially handles the core logic of the app. The images submitted by the user are processed using the Azure Vision API via an Image Analysis client. Azure Vision generates a caption and a confidence score for each image. The caption describes the image. These captions are then passed to the OpenAI Chat Completion API, along with a prompt to craft a 50-word story based on the generated captions, with the context tailored to that of a children's book author.

* create a folder called **utils** in the project home directory. Create a file called **azure\_ai\_services.py** in **utils.**
    
* Update **azure\_ai\_services.py** with the following code.
    

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.core.credentials import AzureKeyCredential
from openai import OpenAI
import os
from django.shortcuts import render
from dotenv import load_dotenv
from azureai.models import Image, Story

# Initialize the Azure and OpenAI clients
load_dotenv()

client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY")
)
cl = ImageAnalysisClient(
    endpoint=os.getenv("VISION_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("VISION_KEY"))
)


def generate_story_from_pictures(image_one, image_two, image_three):
    # Analyze images using Azure Vision API
    image_data_one = image_one.read()
    image_data_two = image_two.read()
    image_data_three = image_three.read()

    result_one = cl.analyze(
        image_data=image_data_one,
        visual_features=["CAPTION", "READ"],
        gender_neutral_caption=True,
    )
    result_two = cl.analyze(
        image_data=image_data_two,
        visual_features=["CAPTION", "READ"],
        gender_neutral_caption=True,
    )
    result_three = cl.analyze(
        image_data=image_data_three,
        visual_features=["CAPTION", "READ"],
        gender_neutral_caption=True,
    )

    # Prepare the prompt using captions from all images
    prompt = f"Write a short story of 50 words that includes these elements: {result_one.caption.text}, " \
             f"{result_two.caption.text}, and {result_three.caption.text}. Generate a title for the story " \
             f"in 5 words or less."

    try:
        # Generate the story using OpenAI API
        completion = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a children's book author skilled in adventure and "
                                              "fiction novels set in 1980."},
                {"role": "user", "content": prompt}
            ]
        )
        generated_story = completion.choices[0].message.content

        # Splitting the response into story and title
        story_parts = generated_story.split("\n\n")

        if len(story_parts) == 2:
            title_text = story_parts[0]
            if title_text.lower().startswith("title:"):
                title_text = title_text[7:].strip()
            story_text = story_parts[1]
        else:
            title_text = "Untitled Story"
            story_text = generated_story

    except Exception as e:
        return render(request, 'pictures_2_stories/create_story.html', {
            'error_message': f"Error generating story: {e}",
        })

    # Save the story and images to the database
    story = Story(generated_story=story_text,
                  title=title_text)
    story.save()

    Image.objects.create(image=image_one, caption=result_one.caption.text, caption_confidence=result_one.caption.confidence, story=story)
    Image.objects.create(image=image_two, caption=result_two.caption.text, caption_confidence=result_two.caption.confidence, story=story)
    Image.objects.create(image=image_three, caption=result_three.caption.text, caption_confidence=result_three.caption.confidence, story=story)

    return story
```

* Update the second `try` block in the `add_sory` method in **views.py** file with the following code. Include the import statement for `generate_story_from_pictures` at the top of the file.
    

```python
from utils.azure_ai_services import generate_story_from_pictures

        try:
            story = generate_story_from_pictures(image_one, image_two, image_three)
        except Exception as e:
            return render(request, 'pictures_2_stories/create_story.html', {
                'error_message': e,
            })
```

* Create a `.env` file in the project root directory and update it with the following info.
    

```python
VISION_KEY=your azure AI vision API KEY
VISION_ENDPOINT=your azure AI vision API endpoint
OPENAI_API_KEY =openAI API key
```

## Azure AI Vision Image Analysis

[Azure AI Vision](https://azure.microsoft.com/en-gb/products/ai-services/ai-vision) is a Microsoft cloud-based service for computer vision. It enables developers to create applications to analyze images, read text, and detect faces with prebuilt image tagging, text extraction with optical character recognition (OCR), and responsible facial recognition.

Azure Vision's image analysis uses AI to identify and categorize content within images automatically. It can detect objects, people, text, and even emotions, making it a powerful tool for understanding and organizing visual data. By processing images through Azure's API, you can extract useful information, such as identifying products in a photo or recognizing handwritten text. This makes Azure Vision valuable for tasks like automating image tagging, enhancing accessibility, and powering smart applications that interact with visual content. Even beginners can start using it by connecting their images to the Azure Vision service and exploring the data it provides.

To get started with image analysis in Vision Studio;

* Create a Computer Vision resource in the Azure Portal ensuring it's in a supported region for the captioning feature. You can use the free tier (F0) initially and upgrade to a paid tier as needed.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723986483856/ad9afa81-d8a4-4546-8181-2d442dc38b93.png align="center")

* After deployment, access the resource to obtain the key and endpoint needed to connect your application. Set the `VISION_KEY` and set the `VISION_ENDPOINT` environment variables in your project `.env` file.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723986839705/aeb248e3-9711-46e3-aaf7-7882f8de7b50.png align="center")

* The `ImageAnalysisClient` is set up using the endpoint and API key stored in environment variables, allowing secure access to Azure Vision services.
    

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient

cl = ImageAnalysisClient(
    endpoint=os.getenv("VISION_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("VISION_KEY"))
)
```

* The `analyze` method is called on the client for each image, specifying visual features like "CAPTION" and "READ" to generate captions and extract text from the images.
    

```python
result_one = cl.analyze(
        image_data=image_data_one,
        visual_features=["CAPTION", "READ"],
        gender_neutral_caption=True,
    )
```

* The `gender_neutral_caption` parameter is set to ensure inclusive captions are generated.
    
* The captions generated by the Azure AI Vision service are extracted from each image using the `analyze` method. When `visual_features=["CAPTION"]` is specified, the service processes the image and returns a descriptive sentence about the content of the image, known as the caption.
    
* The captions extracted from the three images are then used to create a prompt for generating a short story.
    
* The prompt combines the captions, instructing the AI to write a story and generate a title based on the analyzed image content.
    

```python
# Prepare the prompt using captions from all images
    prompt = f"Write a short story of 50 words that includes these elements: {result_one.caption.text}, " \
             f"{result_two.caption.text}, and {result_three.caption.text}. Generate a title for the story " \
             f"in 5 words or less."
```

## OpenAI API for Chat Completion

The OpenAI Chat Completion API is a service that allows developers to generate text responses based on a given prompt using advanced language models like GPT-3.5.

To get an API key, you'll need to sign up for an account on the [OpenAI website](https://platform.openai.com/signup) [and navigate](https://platform.openai.com/signup) to the API keys section in the dashboard. From there, you can generate a new key, which will be used to authenticate your application and access OpenAI's services.

* The `OpenAI` client is initialized using an API key, which is securely retrieved from environment variables via `os.getenv("OPENAI_API_KEY")`. This ensures that sensitive information, like the API key, is not hardcoded in the code.
    

```python
client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY")
)
```

* In the code, we connect to this API using the [`client.chat`](http://client.chat)`.completions.create` method, specifying the model (`gpt-3.5-turbo`) and the messages to guide the AI's response.
    

```python
completion = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a children's book author skilled in adventure and "
                                              "fiction novels set in 1980."},
                {"role": "user", "content": prompt}
            ]
        )
```

* The `messages` parameter is structured as a conversation, where the system message sets the context (e.g., instructing the AI to write as a children's book author), and the user message provides the prompt for story generation.
    
* The API processes the prompt and returns a generated response, which is stored in `completion.choices[0].message.content`.
    

```python
generated_story = completion.choices[0].message.content
```

* To organize the AI's response, we split it into two parts: the title and the story. The title is expected to be in the first part, separated by a double newline (`\n\n`) from the story content.
    
* If the title begins with "Title:", we strip that prefix to clean up the title text. If no title is provided, a default "Untitled Story" is assigned.
    
* The `try-except` block ensures that if any error occurs during the API call, the user is informed with an error message displayed on the story creation page.
    
* This approach allows us to leverage the power of OpenAI's models to generate creative content based on image captions, enhancing the storytelling experience.
    

# **Deploy App with Azure App Service and Azure Database for PostgreSQL**

* Generate a new secret key for your app using the command below in your terminal and save it in the `.env` file. Update the value `SECRET_KEY` in the **settings.py** file to read from the `.env` file.
    

```python
python -c 'import secrets; print(secrets.token_hex())'
```

```python
# .env file
SECRET_KEY=<secret-key>
```

```python
# settings.py file
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY')
```

* Update ALLOWED\_HOSTS to include the app's URL in the settings.py file. Ai runtime, App Service automatically sets the `WEBSITE_HOSTNAME` environment variable to the app's URL.
    

```python
ALLOWED_HOSTS = [os.environ['WEBSITE_HOSTNAME']] if 'WEBSITE_HOSTNAME' in os.environ else []
CSRF_TRUSTED_ORIGINS = ['https://' + os.environ['WEBSITE_HOSTNAME']] if 'WEBSITE_HOSTNAME' in os.environ else ['http://127.0.0.1:8000']
DEBUG = False
```

* To serve static files in the Django production environment update the following in your settings.py file.
    
    * Add **STATIC\_ROOT** to [`settings.py`](http://settings.py). Where **STATIC\_ROOT** is the destination to where static files are copied and from where they are served when deploying the Django app.
        
    * Add a variable for the STATICFILES\_DIRS
        
    * Whitenoise is a Python package that enables a production Django app to serve its own static files that are specified by the Django `STATIC_ROOT` variable. Add a variable for STATICFILES\_STORAGE
        
    * Modify the `MIDDLEWARE` to include Whitenoise
        
        ```python
        MIDDLEWARE = [                                                                   
            'django.middleware.security.SecurityMiddleware',
            # Add whitenoise middleware after the security middleware                             
            'whitenoise.middleware.WhiteNoiseMiddleware',
            # Other values follow
        ]
        
        SESSION_ENGINE = "django.contrib.sessions.backends.cache"
        STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
        STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
        STATICFILES_DIRS = (str(BASE_DIR.joinpath('static')),)
        ```
        
* Sign in to the [Azure portal](https://portal.azure.com/) and create a resource for Azure App Service. Click on **Create a Resource** and search for *web app + database*. Click on **Create**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724523309638/95e66013-9fd2-4a5d-8a90-016e1b54805d.png align="center")

* Set the **Resource Group** as the same one we used for Azure Vision resources. The **Resource Group** for the app lets you group (in a logical container) all the Azure resources needed for the application.
    
* Set the **Region** to run the app physically in the world. Select the closest **Region** to you and your users.
    
* Give your app a **Name** which must be unique across Azure. The name is used as part of the DNS name for your webapp in the form of `https://<app-name>.`[`azurewebsites.net`](http://azurewebsites.net).
    
* The **Runtime stack** for the app. It's where you select the version of Python to use for your app. Select `Python 3.12` as the **Runtime Stack**.
    
* Tick **yes** for Add Azure Cache for Redis.
    
* The **Hosting plan** for the app is the pricing tier that includes the set of features and scaling capacity for your app. Select the **Basic** plan for hobby and research projects.
    
* Click **Review + Create** and then select **Create** after validation. Select the **Go to Resource** button once the deployment completes.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724526401933/8a5fc46c-6f86-4c6b-bbe8-67e06cdb8313.png align="center")

* Environment variables for the app are stored in the **Environment Variables** page under the **Settings** tab. They are added to the app during runtime. verify that `AZURE_POSTGRESQL_CONNECTIONSTRING` and `AZURE_REDIS_CONNECTIONSTRING` are present.
    

* Click on **Add** to include your values for the following variables from your `.env` file. Click **Apply** to update your changes and **Apply** again to save them.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724528626462/2f71ffde-6250-4473-8dcf-7d8d7d28a52b.png align="center")

* Click on **Deployment Center** under the **Deployment** tab.
    
* Ensure that the updated version of your code is on your main branch in GitHub.
    
* Select **GitHub** as your source to deploy and build your code.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724529343472/28de5549-0220-415d-8677-b6c8a48c7f45.png align="center")

* Select your GitHub account as your **Organisation**. Select the **Repository** we created for this project. Set the **Branch** as main. Select **Save**.
    

* Under the Logs tab in the Deployment Center, select **Build/Deploy Logs** in the log item for the deployment run.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724542892819/75fe934b-8ad5-4980-bfbf-22702f05c738.png align="center")

* You can now view GitHub actions running on your repository.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724543065060/291b9341-8810-41ec-a3ae-ca2818af2676.png align="center")

* SSH into the App Service container to run Django database migrations. In the left menu of the App Service page, select **SSH** under the **Development Tools** tab and select **Go**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724581236492/7652961d-34a8-4a31-9e0f-acb7bf6490cf.png align="center")

* Run the command below in the SSH terminal.
    

```python
python manage.py migrate
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724582374599/193fde8b-7467-4e80-b106-bff742e259c5.png align="center")

* Visit the site by clicking on the **Default domain** from the App Service overview page.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724582877007/394c0c5f-5bf3-44e8-82f3-414b283f1fb4.png align="center")

# Resources

* [Quickstart: Image Analysis 4.0](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/image-analysis-client-library-40?tabs=visual-studio%2Clinux&pivots=programming-language-python)