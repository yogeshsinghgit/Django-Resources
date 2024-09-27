## JWT-based authentication with permissions and models with foreign keys using Django Rest Framework (DRF)

Implementing JWT-based authentication with permissions and models with foreign keys using Django Rest Framework (DRF) involves a few key components. Let’s break down the process step-by-step:

1. **Set Up JWT Authentication**
2. **Create Models with Foreign Keys**
3. **Create Views, Serializers, and Permissions**
4. **Implement Permission Classes**
5. **Configure URLs and Authentication Settings**

### Step 1: Set Up JWT Authentication
Install `djangorestframework` and `djangorestframework-simplejwt` to enable JWT support:

```bash
pip install djangorestframework
pip install djangorestframework-simplejwt
```

In your `settings.py`, add the necessary configurations:

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework_simplejwt',
]

# REST Framework settings
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
}
```

Update the URL configuration to include JWT endpoints:

```python
# urls.py
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

This will provide endpoints for obtaining and refreshing JWT tokens.

### Step 2: Create Models with Foreign Keys
Create models with relationships using `ForeignKey` fields. For example, let’s create a blog application where `Author` and `Category` are related to `BlogPost`:

```python
# models.py
from django.contrib.auth.models import User
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class BlogPost(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    author = models.ForeignKey(User, related_name='blog_posts', on_delete=models.CASCADE)
    category = models.ForeignKey(Category, related_name='blog_posts', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

### Step 3: Create Views, Serializers, and Permissions
Create serializers and views to manage these models:

**Serializers:**

```python
# serializers.py
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import BlogPost, Category

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = '__all__'

class BlogPostSerializer(serializers.ModelSerializer):
    author = serializers.ReadOnlyField(source='author.username')
    category = CategorySerializer()

    class Meta:
        model = BlogPost
        fields = ['id', 'title', 'content', 'author', 'category', 'created_at']
```

**Views:**

```python
# views.py
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from .models import BlogPost, Category
from .serializers import BlogPostSerializer, CategorySerializer

class BlogPostViewSet(viewsets.ModelViewSet):
    queryset = BlogPost.objects.all()
    serializer_class = BlogPostSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class CategoryViewSet(viewsets.ModelViewSet):
    queryset = Category.objects.all()
    serializer_class = CategorySerializer
    permission_classes = [IsAdminUser]
```

### Step 4: Implement Permission Classes
You can define custom permission classes to control access. For example, you might want to restrict certain operations to only the author of the blog post:

```python
# permissions.py
from rest_framework.permissions import BasePermission

class IsAuthorOrReadOnly(BasePermission):
    """
    Custom permission to only allow authors of a blog post to edit it.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True

        # Write permissions are only allowed to the author of the blog post
        return obj.author == request.user
```

Use this permission in your `BlogPostViewSet`:

```python
# views.py
class BlogPostViewSet(viewsets.ModelViewSet):
    queryset = BlogPost.objects.all()
    serializer_class = BlogPostSerializer
    permission_classes = [IsAuthenticated, IsAuthorOrReadOnly]  # Apply custom permission

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

### Step 5: Configure URLs and Authentication Settings
Include the API endpoints for your models in the URL configuration:

```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import BlogPostViewSet, CategoryViewSet

router = DefaultRouter()
router.register(r'blogposts', BlogPostViewSet)
router.register(r'categories', CategoryViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

### Testing JWT Authentication
1. Obtain a JWT token using the `api/token/` endpoint by passing username and password.
2. Use the obtained token in the `Authorization` header as `Bearer <your_token>` to access protected endpoints like `/api/blogposts/`.
3. Test different endpoints and permissions (e.g., only admins can manage categories).

With this setup, you have:
- JWT authentication integrated.
- Models with foreign keys.
- Permissions applied based on user roles and object relationships.
