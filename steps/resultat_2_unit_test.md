## Steps/Commands
>Note: Please 'cd' into the root directory and fire up your virtual environment!
In this module, we will write some unit tests to test our previous contact us endpoint.

1) Unit tests - Copy the following code into /core/tests.py
from . models import Contact
from rest_framework.test import APIClient
from rest_framework.test import APITestCase
from rest_framework import status



class ContactTestCase(APITestCase):

    """
    Test suite for Contact
    """
    def setUp(self):
        self.client = APIClient()
        self.data = {
            "name": "Billy Smith",
            "message": "This is a test message",
            "email": "billysmith@test.com"
        }
        self.url = "/contact/"

    def test_create_contact(self):
        '''
        test ContactViewSet create method
        '''
        data = self.data
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(Contact.objects.count(), 1)
        self.assertEqual(Contact.objects.get().title, "Billy Smith")

    def test_create_contact_without_name(self):
        '''
        test ContactViewSet create method when name is not in data
        '''
        data = self.data
        data.pop("name")
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_create_contact_when_name_equals_blank(self):
        '''
        test ContactViewSet create method when name is blank
        '''
        data = self.data
        data["name"] = ""
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

    def test_create_contact_without_message(self):
        '''
        test ContactViewSet create method when message is not in data
        '''
        data = self.data
        data.pop("message")
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_create_contact_when_message_equals_blank(self):
        '''
        test ContactViewSet create method when message is blank
        '''
        data = self.data
        data["message"] = ""
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

    def test_create_contact_without_email(self):
        '''
        test ContactViewSet create method when email is not in data
        '''
        data = self.data
        data.pop("email")
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_create_contact_when_email_equals_blank(self):
        '''
        test ContactViewSet create method when email is blank
        '''
        data = self.data
        data["email"] = ""
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

    def test_create_contact_when_email_equals_non_email(self):
        '''
        test ContactViewSet create method when email is not email
        '''
        data = self.data
        data["email"] = "test"
        response = self.client.post(self.url, data)
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

2) Run tests - Go ahead and run our new tests in  docker desktop api container exec page

```
python manage.py test
```
3) je n'ai pas exécuté le point 3 du module

4) Check database - You can now use the Django shell to check the database for our new contact entry. Open Django shell with the following command in docker desktop exec api container
```
python manage.py shell
```
Now use the following command and check the database for our new entry
```
from core.models import Contact
c = Contact.objects.last()
c.title
```
>Note: on peut aussi remplacer c.title juste par c