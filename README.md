# djangoclass
- Create a Django project called AuthenticationProject:

    ```shell
    django-admin startproject AuthenticationProject
    ```

- Create an app called Core:

    ```shell
    python manage.py startapp Core
    ```

- Open the project in your code editor.

- Create a templates folder and register it in the project's settings.

- Register the app in the project's settings.

- Create URLs for the app and register them in the project's URLs.

- Setup static files in `settings.py`:

    ```python

    import os # at top of file

    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR,  'staticfiles')
    STATICFILES_DIRS = (os.path.join(BASE_DIR, 'static'), )
    ```

### 5. Getting Template Files from GitHub

   - create the following HTML templates :
     - `index.html`
     - `login.html`
     - `register.html`
    

### 6. Making required imports

- Head to your views.py file and import the following:

    ```python
    from django.shortcuts import render, redirect
    from django.contrib.auth.models import User
    from django.contrib.auth import authenticate, login, logout
    from django.contrib.auth.decorators import login_required
    from django.contrib import messages
    from django.conf import settings
    from django.core.mail import EmailMessage
    from django.utils import timezone
    from django.urls import reverse
    from .models import *
    ```

### 7. Create a super user

- Create a super user:

```python
python manage.py createsuperuser
```

- login to admin dashboard with credentials:
    `127.0.0.1:8000/admin`

### 8. Creating Home, Register, & Login Views

- Create home view:

    ```python
    def Home(request):
        return render(request, 'index.html')
    ```

- Create two new views for `Register` and `Login`:

    ```python
    def RegisterView(request):
        return render(request, 'register.html')

    def LoginView(request):
        return render(request, 'login.html')
    ```

- Map views to urls:

    ```python
    path('', views.Home, name='home'),
    path('register/', views.RegisterView, name='register'),
    path('login/', views.LoginView, name='login'),
    ```

### 9. Working on Register View

- Change static file links in all files: 

    ```html
    <link rel="stylesheet" href="{% static 'style.css' %}">
    ```

- Head to register.html and give input fields a name attribute & add csrf_token and change the login url:

    ```html
    <form method="POST">

        {% csrf_token %}
      
        <div class="txt_field">
            <input type="text" required name="first_name">
            <span></span>
            <label>First Name</label>
          </div>

          <div class="txt_field">
            <input type="text" required name="last_name">
            <span></span>
            <label>Last Name</label>
          </div>

        <div class="txt_field">
          <input type="text" required name="username">
          <span></span>
          <label>Username</label>
        </div>

        <div class="txt_field">
            <input type="email" required name="email">
            <span></span>
            <label>Email</label>
          </div>

        <div class="txt_field">
          <input type="password" required name="password">
          <span></span>
          <label>Password</label>
        </div>    

        <!-- <div class="pass">Forgot Password?</div> -->
        <input type="submit" value="Register">
        <div class="signup_link">
          Already have an account? <a href="{% url 'login' %}">Login</a>
        </div>
      </form>
    ```

- In `RegisterView` view Check for incoming form submission and grab user data:

    ```python
    if request.method == 'POST:

        # getting user inputs from frontend
        first_name = request.POST.get('first_name')
        last_name = request.POST.get('last_name')
        username = request.POST.get('username')
        email = request.POST.get('email')
        password = request.POST.get('password')
    ```

- validate the data provided:

    - create flag for error

        ```python
        user_data_has_error = False
        ```
    - validate email and username:

        ```python
        # make sure email and username are not being used

        if User.objects.filter(username=username).exists():
            user_data_has_error = True
            messages.error(request, 'Username already exists')

        if User.objects.filter(email=email).exists():
            user_data_has_error = True
            messages.error(request, 'Email already exists')
        ```
    - validate password length:

        ```python
        # make aure password is at least 5 characters long
        if len(password) < 5:
            user_data_has_error = True
            messages.error(request, 'Password must be at least 5 characters')
        ```

- Create a new user if there are no errors and redirect to the login page. Else redirect back to the register page with errors

    ```python
    if not user_data_has_error:
        new_user = User.objects.create_user(
            first_name = first_name,
            last_name = last_name,
            email = email,
            username = username,
            password = password
        )
        messages.success(request, 'Account created. Login now')
        return redirect('login')
    else:
        return redirect('register')
    ```

- Display incoming messages in `register.html`, `login.html`, `forgot_password.html`, and `reset_password.html` files:

    ```html
    {% if messages %}
        {% for message in messages %}
            {% if messages.tags == 'error' %}
                <center><h4 style="color: firebrick;">{{message}}</h4></center>
            {% else %}
                <center><h4 style="color: dodgerblue;">{{message}}</h4></center>
            {% endif %}
        {% endfor %}
    {% endif %}

    <form method="POST">
        ...
    </form>
    ```

- Test code to see if users can now register

