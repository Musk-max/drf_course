## Steps/Commands

>Note: Please 'cd' into the root directory and fire up your virtual environment!
We need models, routers, serializers and view-sets/api-view! Let's not waist any time...

1) Model - Go ahead and open /ecommerce/models.py and paste in the following code.
```
from django.db import models
from django.contrib.auth.models import User
from utils.model_abstracts import Model
from django_extensions.db.models import (
    TimeStampedModel,
    ActivatorModel,
    TitleSlugDescriptionModel
)


class Item(
    TimeStampedModel,
    ActivatorModel ,
    TitleSlugDescriptionModel,
    Model):

    """
    ecommerce.Item
    Stores a single item entry for our shop
    """

    class Meta:
        verbose_name = 'Item'
        verbose_name_plural = 'Items'
        ordering = ["id"]

    def __str__(self):
        return self.title
    
    stock = models.IntegerField(default=1)
    price = models.IntegerField(default=0)

    def amount(self):
        #converts price from pence to pounds
        amount = float(self.price / 100)
        return amount

    def manage_stock(self, qty):
        #used to reduce Item stock
        new_stock = self.stock - int(qty)
        self.stock = new_stock
        self.save()


    def check_stock(self, qty):
        #used to check if order quantity exceeds stock levels
        if int(qty) > self.stock:
            return False
        return True

    def place_order(self, user, qty):
        #used to place an order
        if self.check_stock(qty):
            order = Order.objects.create(
                item = self, 
                quantity = qty, 
                user= user)
            self.manage_stock(qty)
            return order
        else:
            return None




class Order(
    TimeStampedModel,
    ActivatorModel ,
    Model):
    """
    ecommerce.Order
    Stores a single order entry, related to :model:`ecommerce.Item` and
    :model:`auth.User`.
    """
    class Meta:
        verbose_name = 'Order'
        verbose_name_plural = 'Orders'
        ordering = ["id"]
    
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=True, blank=True)
    item = models.ForeignKey(Item, null=True, blank=True, on_delete=models.CASCADE)
    quantity = models.IntegerField(default=0)

    def __str__(self):
        return f'{self.user.username} - {self.item.title}'

```
2) Serializers - Go ahead and create a new file in /ecommerce and call it serializers.py. Use the following code.
```

from collections import OrderedDict
from .models import Item , Order
from rest_framework_json_api import serializers
from rest_framework import status
from rest_framework.exceptions import APIException




class NotEnoughStockException(APIException):
    status_code = status.HTTP_400_BAD_REQUEST
    default_detail = 'There is not enough stock'
    default_code = 'invalid'




class ItemSerializer(serializers.ModelSerializer):
    
    class Meta:
        model = Item
        fields = (
            'title',
            'stock',
            'price',
        )




class OrderSerializer(serializers.ModelSerializer):

    item = serializers.PrimaryKeyRelatedField(queryset = Item.objects.all(), many=False)
    
    class Meta:
        model = Order
        fields = (
            'item',
            'quantity',
        )

    def validate(self, res: OrderedDict):
        '''
        Used to validate Item stock levels
        '''
        item = res.get("item")
        quantity = res.get("quantity")
        if not item.check_stock(quantity):
            raise NotEnoughStockException
        return res
```
3) Register - Let's go ahead and register our new models to the built in admin pages. Open /ecommerce/admin.py and paste in the following code.

```
from django.contrib import admin
from . import models


@admin.register(models.Item)
class ItemAdmin(admin.ModelAdmin):
    list_display = ('id', 'title')


@admin.register(models.Order)
class OrderAdmin(admin.ModelAdmin):
    list_display = ('id', 'item')
```
4) Migrate - We can now migrate the models to the database. Use the following code in the docker api container.
```
python manage.py makemigrations
python manage.py migrate
```
5) then update your online admin page to see the results