1&#8203;. Install [pyrollbar](https://github.com/rollbar/pyrollbar) with [pip](http://pip.readthedocs.org/en/stable/quickstart/):

```shell
pip install rollbar
```

2&#8203;. In your ``settings.py``, add ``'rollbar.contrib.django.middleware.RollbarNotifierMiddleware'`` as the last item in ``MIDDLEWARE_CLASSES``:

```python
MIDDLEWARE_CLASSES = (
    # ... other middleware classes ...
    'rollbar.contrib.django.middleware.RollbarNotifierMiddleware',
)
```

3&#8203;. Add the following in ``settings.py``:

```python
ROLLBAR = {
    'access_token': '{{ server_access_token }}',
    'environment': 'development' if DEBUG else 'production',
    'root': BASE_DIR,
}
import rollbar
rollbar.init(**ROLLBAR)
```

4&#8203;. Send a test message:

```shell
python manage.py shell
>>> import rollbar
>>> rollbar.report_message("Hello world")
```
