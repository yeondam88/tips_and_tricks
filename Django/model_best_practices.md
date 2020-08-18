# Model Best Practice

1. Break up Apps with too many models (No more than five to ten models per app.)

2. Try to avoid multi-table inferitance

3. Use abstract model for such as TimeStampModel

```python
from django.db import models

class TimeStampedModel(models.Model):
    """
    An abstract base class model that provides self-updating
    ``created`` and ``modified`` fields.
    """
    created = models.DateTimeField(auto_now_add=True)
    modified = models.DateTimeField(auto_now=True)
  
    class Meta:
        abstract = True # Setting this model to Abstract model (This does not create a table)
```

## Django Model Design

1. Start Normalized.  

- When you're designing your Django models, always start off normalized. No model should contain data already stored in another model.

2. Cache Before Denormalizing

3. Denormalize Only if Absolutely Needed

4. When to Use Null and Blank

5. When to Use Binary Field
  - e.g. Compressed data; the type of data Sentry stores as a BLOB, but is required to base64-encode due to legacy issues.

| Field Type                                                                         | Setting null=True                                                                                                                                                                         | Setting blank=True                                                                                                                                                                                                             |
|------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CharField, TextField, SlugField, EmailField, CommaSeparatedIntegerField, UUIDField | Okay if you also have set both unique=True and blank=True. In this situation, null=True is required to avoid unique constraint violations when saving multiple objects with blank values. | Okay if you want the corresponding form widget to accept empty values. If you set this, empty values are stored as NULL in the database if null=True and unique=True are also set. Otherwise, they get stored as empty strings |
| FileField, ImageField                                                              | Don’t do this. Django stores the path from MEDIA_ROOT to the file or to the image in a CharField, so the same pattern applies to FileFields.                                              | Okay. The same pattern for CharField applies here.                                                                                                                                                                             |
| BooleanField                                                                       | Okay                                                                                                                                                                                      | Default is blank=True                                                                                                                                                                                                          |
| IntegerField, FloatField, DecimalField, DurationField, etc                         | Okay if you want to be able to set the value to NULL in the database.                                                                                                                     | Okay if you want the corresponding form widget to accept empty values. If so, you will also want to set null=True.                                                                                                             |
| DateTimeField, DateField, TimeField, etc.                                          | Okay if you want to be able to set the value to NULL in the database.                                                                                                                     | Okay if you want the corresponding form widget to accept empty values, or if you are using auto_now or auto_now_add. If it’s the former, you will also want to set null=True.                                                  |
| ForeignKey, OneToOneField                                                          | Okay if you want to be able to set the value to NULL in the database.                                                                                                                     | Okay if you want the corresponding form widget (e.g. the select box) to accept empty values. If so, you will also want to set null=True.                                                                                       |
| ManyToManyField                                                                    | Null has no effect                                                                                                                                                                        | Okay if you want the corresponding form widget(e.g. the select box) to accept empty values                                                                                                                                     |
| GenericIPAddressField                                                              | Okay if you want to be able to set the value to NULL in the database.                                                                                                                     | Okay if you want to make the corresponding field widget accept empty values. If so, you will also want to set null=True.                                                                                                       |
| JSONField                                                                          | Okay                                                                                                                                                                                      | Okay                                                                                                                                                                                                                           |

6. Try to Avoid Using Generic Relations
  - Reduction in speed of queries due to lack of indexing between models
  - Danger of data corruption as a table can refer to another against a non-existent record
  
7. Make Choices and Sub-Choices Model Constants
8. Using Enumeration Types for Choices
```python
from django.db import models

class IceCreamOrder(models.Model):
    class Flavors(models.TextChoices):
        CHOCOLATE = 'ch', 'Chocolate'
        VANILLA = 'vn', 'Vanilla'
        STRAWBERRY = 'st', 'Strawberry'
        
    flavor = models.CharField(max_length=2, choices=Flavors.choices)
    
In Django shell
>>> from orders.models import IceCreamOrder
>>> IceCreamOrder.objects.filter(flavor=IceCreamOrder.Flavors.CHOCOLATE)
[<icecreamorder: 35>, <icecreamorder: 42>, <icecreamorder: 49>]
```
> Named groups are not possible with enumeration types.
> If we want other types besides str and int, we have to define those ourselves

9. Model Managers
Model managers are said to act on the full set of all possible instances of this model class to restrict the ones you want to
work with.
```python
from django.db import models
from django.utils import timezone

class PublishedManager(models.Manager):
    
    def published(self, **kwargs):
        return self.filter(pub_date__lte=timezone.now(), **kwargs)

class FlavorReview(models.Model):
    review = models.CharField(max_length=255)
    pub_date = models.DateTimeField()
    
    # add our custom model manager
    objects = PublishedManager()

# Now, if we first want to display a count of all of the ice cream flavor reviews, and then a count of just published ones,
we can do the following:
>>> from reviews.models import FlavorReview
>>> FlavorReview.objects.count()
35
>>> FlavorReview.objects.published().count()
31
```

10. Stateless Heloper Functions
- The downside is that the functions are stateless, hence all arguments have to be passed

###### This one of sections from Two scoops of Django 3.x book
