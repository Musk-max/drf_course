## Steps/Commands

>Note: Please 'cd' into the root directory and fire up your virtual environment!
1) Unit tests - Copy the following code into /ecommerce/tests.py

```
from django.contrib.auth.models import User
from ecommerce.models import Item, Order
from rest_framework.authtoken.models import Token
from rest_framework.test import APIClient
from rest_framework.test import APITestCase
from rest_framework import status


class EcommerceTestCase(APITestCase):
    """
    Test suite for Items and Orders
    """
    def setUp(self):

        Item.objects.create(title= "Demo item 1",description= "This is a description for demo 1",price= 500,stock= 20)
        Item.objects.create(title= "Demo item 2",description= "This is a description for demo 2",price= 700,stock= 15)
        Item.objects.create(title= "Demo item 3",description= "This is a description for demo 3",price= 300,stock= 18)
        Item.objects.create(title= "Demo item 4",description= "This is a description for demo 4",price= 400,stock= 14)
        Item.objects.create(title= "Demo item 5",description= "This is a description for demo 5",price= 500,stock= 30)
        self.items = Item.objects.all()
        self.user = User.objects.create_user(
            username='testuser1', 
            password='this_is_a_test',
            email='testuser1@test.com'
        )
        Order.objects.create(item = Item.objects.first(), user = User.objects.first(), quantity=1)
        Order.objects.create(item = Item.objects.first(), user = User.objects.first(), quantity=2)
        
        #The app uses token authentication
        self.token = Token.objects.get(user = self.user)
        self.client = APIClient()
        
        #We pass the token in all calls to the API
        self.client.credentials(HTTP_AUTHORIZATION='Token ' + self.token.key)


    def test_get_all_items(self):
        '''
        test ItemsViewSet list method
        '''
        self.assertEqual(self.items.count(), 5)
        response = self.client.get('/item/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_get_one_item(self):
        '''
        test ItemsViewSet retrieve method
        '''
        for item in self.items:
            response = self.client.get(f'/item/{item.id}/')
            self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_order_is_more_than_stock(self):
        '''
        test Item.check_stock when order.quantity > item.stock
        '''
        for i in self.items:
            current_stock = i.stock
            self.assertEqual(i.check_stock(current_stock + 1), False)

    def test_order_equals_stock(self):
        '''
        test Item.check_stock when order.quantity == item.stock
        '''
        for i in self.items:
            current_stock = i.stock
            self.assertEqual(i.check_stock(current_stock), True)

    def test_order_is_less_than_stock(self):
        '''
        test Item.check_stock when order.quantity < item.stock
        '''
        for i in self.items:
            current_stock = i.stock
            self.assertTrue(i.check_stock(current_stock - 1), True)
    
    def test_create_order_with_more_than_stock(self):
        '''
        test OrdersViewSet create method when order.quantity > item.stock
        '''
        for i in self.items:
            stock = i.stock
            data = {"item": str(i.id), "quantity": str(stock+1)}
            response = self.client.post(f'/order/', data)
            self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)

    def test_create_order_with_less_than_stock(self):
        '''
        test OrdersViewSet create method when order.quantity < item.stock
        '''
        for i in self.items:
            data = {"item": str(i.id), "quantity": 1}
            response = self.client.post(f'/order/',data)
            self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_create_order_with_equal_stock(self):
        '''
        test OrdersViewSet create method when order.quantity == item.stock
        '''
        for i in self.items:
            stock = i.stock
            data = {"item": str(i.id), "quantity": str(stock)}
            response = self.client.post(f'/order/',data)
            self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_get_all_orders(self):
        '''
        test OrdersViewSet list method
        '''
        self.assertEqual(Order.objects.count(), 2)
        response = self.client.get('/order/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        

    def test_get_one_order(self):
        '''
        test OrdersViewSet retrieve method
        '''
        orders = Order.objects.filter(user = self.user)
        for o in orders:
            response = self.client.get(f'/order/{o.id}/')
            self.assertEqual(response.status_code, status.HTTP_200_OK)
```
2) Run tests - You can run our new tests with the following command in docker api container.
```
python manage.py test
```
3)  Call our endpoints - Here are the requests we can make to our new endpoints in our docker app container exec page.
> This retrieves the auth token for bobby

http post http://api:8000/api-token-auth/ username=bobby password=1234

> This will retrieve all items

http http://api:8000/item/ 'Authorization: Token 6801789df1027cb5e5af6d814aa2a92458c0a285'

> This will retreive a single item

http http://api:8000/item/e8961813-f537-4cde-90b5-c5330df515a0/ 'Authorization: Token 6801789df1027cb5e5af6d814aa2a92458c0a285'

> This retrieve all orders

http http://api:8000/order/ 'Authorization: Token 6801789df1027cb5e5af6d814aa2a92458c0a285'

> This will place some orders for item id = **your_item_uuid** quantity = 1

http http://api:8000/order/ 'Authorization: Token 6801789df1027cb5e5af6d814aa2a92458c0a285' item="e8961813-f537-4cde-90b5-c5330df515a0" quantity="1"

http http://api:8000/order/ 'Authorization: Token 6801789df1027cb5e5af6d814aa2a92458c0a285' item="e8961813-f537-4cde-90b5-c5330df515a0" quantity="5"