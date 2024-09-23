---
title: "Update Nested Serializers Fields with Django REST Framework"
datePublished: Sun Aug 28 2022 17:17:44 GMT+0000 (Coordinated Universal Time)
cuid: cl7dlhdcs02uo1cnvarmo4ufk
slug: update-nested-serializers-fields-with-django-rest-framework
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1661682633475/O5deVPUdr.png
tags: postman, serialization, python, django, django-rest-framework

---

# Introduction
Hello all 💙 😁 !

If you have attempted to build a web API endpoint for CRUDing (creating, retrieving, updating and deleting) data objects in a database using [Django REST framework](https://www.django-rest-framework.org/), you will realise that the object fields related to other objects in the database are not CRUDed by default 😰 . They have to be serialized appropriately 💪.

In this tutorial, we will build an employee data app that stores information (employee id, first name, last name, employment date and certifications) on every employee on the app as an object in the database. Since every employee, may possess one or more certifications (or even no certifications) we will use a [Many-to-One relationship](https://docs.djangoproject.com/en/4.1/topics/db/examples/many_to_one/) (also known as a foreign key relationship) to map every employee to their certifications in the database. 

📌 See the full code of [GitHub](https://github.com/UffaModey/employee-data)

# Learning Objectives

1. Create Django database models with a foreign-key relationship
2. Create API end-points for CRUDing data in the database
3. Test API end-points on Postman

# Prerequisites 💥 💥 
This demo assumes that you are familiar with the following;

1. Setting up a Django app and deploying it locally. Visit the [Django docs](https://docs.djangoproject.com/en/4.0/intro/tutorial01/) for a detailed tutorial on the process.
2. Using a code editor of your choice to open and edit code files. I am using [Pycharm](https://www.jetbrains.com/pycharm/).
3. Running Django CLI commands from the relevant project directories to perform database migrations.
4. Setting up a virtual environment on local machine to install Django and Django REST framework used for the tutorial.

# Development 💻 

## Step 1: Building Django models and viewing on the Django admin site
The data fields that are stored for each employee are shown in the models.py file for the app below

```
import uuid
from django.utils.translation import gettext_lazy as _

from django.db import models


class Employees(models.Model):
    """
        Model representing Employees
    """
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    first_name = models.CharField(_('first_name'), max_length=255, blank=True, null=True)
    last_name = models.CharField(max_length=255, blank=True, null=True)
    employment_date = models.DateField(blank=True, null=True)

    class Meta:
        verbose_name_plural = "Employees"
        ordering = ['first_name']

    def __str__(self):
        return self.first_name


class Certifications(models.Model):
    """
        Model representing Employees
    """
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(_('name'), max_length=255)
    expiry_date = models.DateField(blank=True, null=True)

    # relations
    employees = models.ForeignKey(Employees, on_delete=models.SET_NULL, blank=True, null=True,
                                  related_name='certifications')


    class Meta:
        verbose_name_plural = "Certifications"
        ordering = ['name']

    def __str__(self):
        return self.name
```

Run the following commands on your terminal to implement the changes in the database.

```
python manage.py makemigrations
python manage.py migrate
```
Create a superuser account that you can use to access the admin site using the command below. 
```
python manage.py createsuperuser
```
Register your models on the admin.py file so that the tables that are created for them in the database are visible to you on the admin site. 
```
from django.contrib import admin

from . import models

@admin.register(models.Certifications)
class CertificationsAdmin(admin.ModelAdmin):
    list_display = ["id", "name", "expiry_date"]


@admin.register(models.Employees)
class EmployeesAdmin(admin.ModelAdmin):
    list_display = ["id", "first_name", "last_name", "employment_date"]

```
Open the Django admin site for your app to view the database models 🚀 🚀 🚀. 

![Django_admin_site.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661102939298/LPK5Vqlhn.png align="left")

## Step 2: Developing API endpoints to CRUD the app data using Django REST Framework
- Ensure that Django REST framework is included in the settings.py file.
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'employee',
    'rest_framework'
]
```
- Update your urls.py file with the relevant CRUD endpoint views for your models.

```
from django.contrib import admin
from django.urls import path

from employee.api import (
    CertificationsListView,
    EmployeesListCreateView,
    EmployeesRetrieveUpdateDeleteView
)

urlpatterns = [
    path('admin/', admin.site.urls),

    # Certifications
    path("certifications", CertificationsListView.as_view(), name="certifications"),

    # Employees
    path("", EmployeesListCreateView.as_view(), name="employees"),
    path("<uuid:pk>", EmployeesRetrieveUpdateDeleteView.as_view(), name="employees-details"),
]
```

- Django REST framework uses generic views to enable users to CRUD endpoints quite easily. Further info about using them can be found [on the DRF generic views docs](https://www.django-rest-framework.org/api-guide/generic-views/). Create an api.py file in your app folder and update it with the following DRF class based views for the app.

```
from rest_framework import generics

from .models import (
    Certifications,
    Employees
)

from .serializers import (
    CertificationsSerializer,
    EmployeesSerializer,
)


class CertificationsListView(generics.ListAPIView):
    """
        List all Certifications
    """
    serializer_class = CertificationsSerializer
    queryset = Certifications.objects.all()


class EmployeesListCreateView(generics.ListCreateAPIView):
    """
        List, Create all Employees
    """
    serializer_class = EmployeesSerializer
    queryset = Employees.objects.all()


class EmployeesRetrieveUpdateDeleteView(generics.RetrieveUpdateDestroyAPIView):
    """
        Retrieve, Update or Delete an Employee
    """
    serializer_class = EmployeesSerializer

    def get_queryset(self):
        return Employees.objects.all()

```
- Create another file in the app folder called serializers.py file.

```
from rest_framework import serializers

from .models import (
    Certifications,
    Employees
)

class CertificationsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Certifications
        fields = ['id', 'name', 'expiry_date']

class EmployeesSerializer(serializers.ModelSerializer):
    class Meta:
        model = Employees
        fields = "__all__"
```

## Step 3: Using Postman to CRUD data on the app
[Postman](https://www.postman.com/) is an excellent tool for testing API endpoints. With Postman, a user is able to provide data in JSON format that can be used to create, retrieve, update and delete data used on the app. 

- To test our endpoint for retrieving employee objects, create an employee on the Django admin side that has a certification related to them.


![Screenshot 2022-08-21 at 21.02.38.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661112164082/3DYYPUHyM.png align="left")

![Screenshot 2022-08-21 at 21.01.44.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661112108274/EY0P7MReP.png align="left")

### Retrieving employee objects with their related certifications
- Send a GET request to the API endpoint for listing the employees in the database.

![Screenshot 2022-08-27 at 23.48.34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661640518763/_XXnXDKb8.png align="left")

The certifications for the employees are not returned in the request response. This is because the certifications fields for the employee has not been properly serialized. Update the employee serializer class  to include certifications and set read_only and many to True.

```
class EmployeesSerializer(serializers.ModelSerializer):
    certifications = CertificationsSerializer(many=True, read_only=True)

    class Meta:
        model = Employees
        fields = "__all__"
```
- Now sending a GET request to the API endpoint for listing the employees in the database will give the following response.

![Screenshot 2022-08-27 at 23.49.57.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661640601902/rwtk0O0Di.png align="left")

- Similarly, an API request to retrieve data for a specific employee will give the response below

![Screenshot 2022-08-28 at 00.04.11.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661641455149/0UiCI-6So.png align="left")

### Creating employee objects and their related certifications
- The image below shows how a POST request is sent to the URL endpoint to create an employee object that is visible on the Django admin site.


![Screenshot 2022-08-28 at 00.13.04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661641990832/D3ByWSwyZ.png align="left")

- Although the object for the employee is created, the related certification for the employee is not created by default. To create it, we have to override the create method in the employee serializer class.

```
class EmployeesSerializer(serializers.ModelSerializer):
    certifications = CertificationsSerializer(many=True, read_only=True)

    @transaction.atomic
    def create(self, validated_data):
        certifications_data = self.initial_data.get('certifications')

        instance = super().create(validated_data)
        if certifications_data:
            Certifications.objects.bulk_create([
                Certifications(
                    name=certification['name'],
                    expiry_date=certification['expiry_date'],
                    employees_id=instance.id
                ) for certification in certifications_data if certification.get('name')
            ])
        # save the instance
        instance.save()
        return instance
    class Meta:
        model = Employees
        fields = "__all__"

``` 
- Now a new employee and their related certifications may be created using a POST request to the API endpoint.

![Screenshot 2022-08-28 at 00.30.00.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661643004476/WrApXe9JZ.png align="left")

### Updating employee objects and their related certifications in the nested fields
To make changes to the recently created employee, we send PATCH request to the http://127.0.0.1:8000/<uuid:pk> API endpoint view. In this tutorial, we desire to perform the following changes;

- update the name of the existing certification for an employee
- create a new certification for the employee
- update the name of the employee

![Screenshot 2022-08-28 at 00.46.21.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661643985382/oIUa26_q1.png align="left")

The certification data for the employees is not updated by default. To update the certification data, we need to override the update method on the employees serializer class.

```
@transaction.atomic
    def update(self, instance, validated_data):
        certifications_data = self.initial_data.get('certifications')
        if certifications_data:
            instance.certifications.clear()
            for certification in certifications_data:
                certification_id = certification.get("id")
                if certification_id:
                    cert_obj = Certifications.objects.get(id=certification_id)
                    cert_obj.name = certification.get("name")
                    cert_obj.expiry_date = certification.get("expiry_date")
                    cert_obj.save()
                    instance.certifications.add(cert_obj)
                else:
                    Certifications.objects.create(
                        name=certification.get("name"),
                        expiry_date=certification.get("expiry_date"),
                        employees_id=instance.id
                    )

        instance = super().update(instance, validated_data)
        instance.save()
        return instance
``` 

![Screenshot 2022-08-28 at 00.53.49.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661644433953/JlALdwjXx.png align="left")

# Conclusion 💃 💃
In this tutorial, we have highlighted the following;

1. Creating Django database models with Foreign Key relations.
2. Creating API POST, PATCH, GET, and DEL endpoints to create, update, retrieve and delete objects from the database. We also showcased how Postman may be used to make these requests.
3. Overriding the create and update method of the serializer class to make changes to the nested fields within an object.

# Next Steps

- View the full project code on [GitHub](https://github.com/UffaModey/employee-data).
- Test sending a DELETE request on Postman to delete one of the employee objects that have been created? What happens to the certifications that are related to this employee?
- Comments, questions or suggestions related to this demo? Please share them with me. I'll be delighted to hear from you.

Cheers! 🎊 🍻 🎊 🍻 🎊 🍻































