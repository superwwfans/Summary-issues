7[down vote]()accepted

The Blueprint equivalent is called [`@Blueprint.before_app_first_request`](http://flask.pocoo.org/docs/0.10/api/#flask.Blueprint.before_app_first_request):

```
@my_blueprint.before_app_first_request
def init_my_blueprint():
    print 'yes'

```

The name reflects that it is called before *any* request, not just a request specific to this blueprint.

There is no hook for running code for just the first request to be handled by your blueprint. You can simulate that with a [`@Blueprint.before_request` handler](http://flask.pocoo.org/docs/0.10/api/#flask.Blueprint.before_request) that tests if it has been run yet:

```
from threading import Lock

my_blueprint._before_request_lock = Lock()
my_blueprint._got_first_request = False

@my_blueprint.before_request
def init_my_blueprint():
    if my_blueprint._got_first_request:
        return
    with my_blueprint._before_request_lock:
        if my_blueprint._got_first_request:
            return 
        my_blueprint._got_first_request = True

        # first request, execute what you need.
        print 'yes'
```

This mimics what Flask does here; locking is needed as separate threads could race to the post to be first.