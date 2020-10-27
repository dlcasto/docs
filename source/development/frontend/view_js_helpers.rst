==============================
View construction (and tools)
==============================

Although most of our code base is being processed server side, some things just require interaction on the
clients machine for a fluent user experience.

In this chapter we will try to explain some of the components we use when designing pages and how pages are usually constructed.

--------------------------
Layout
--------------------------

To ease reading of volt templates, we recommend using a fixed layout when creating templates.
The base of our rendered page always contains the standard `layout <https://github.com/opnsense/core/blob/master/src/opnsense/mvc/app/views/layouts/default.volt>`__
which is hooked via our standard frontend controller.

Below you will find the sections and their order, which we will describe briefly.

.. code-block:: html

    {#
      {1} Copyright notice
    #}
    <script>
    $( document ).ready(function() {
      {2} UI code
    });
    </script>
    {3} page html
    {{ partial("layout_partials/base_dialog",...)}}  {4} dialog forms (see getForm())


#.   The copyright block, 2 clause BSD with the authors on top
#.   Javascript code which belongs to this page
#.   HTML code, usually starts with some :code:`<div>` containers and uses standard Bootstrap 3 layouting
#.   When forms are used, these are placed last, these will be generated to the client as standard html code


----------------------------
ajaxCall
----------------------------

:code:`ajaxCall(url, sendData, callback)` is a wrapper around jQuery's :code:`$.ajax` call preset to a :code:`POST` type
request and wrapping the sendData into a json object.
The :code:`callback` function will be called with the data and status received from the endpoint.



.. code-block:: javascript
    :name: ajaxCall
    :caption: example usage

    ajaxCall('/api/monit/status/get/xml', {}, function(data, status) {
        console.log(data)
    });


----------------------------
ajaxGet
----------------------------

:code:`ajaxGet(url,sendData,callback)` is also a wrapper around jQuery's :code:`$.ajax` call, but for a :code:`GET` type
request.

.. code-block:: javascript
    :name: ajaxGet
    :caption: example usage

    ajaxGet('/api/diagnostics/interface/getInterfaceNames', {}, function(data, status) {
        console.log(data);
    });


----------------------------
mapDataToFormUI
----------------------------

The :code:`mapDataToFormUI(data_get_map, server_params)` can be used to map data retrieved from a controller to a
form in the browser.

This function accepts two parameters, data_get_map contains a mapping between form id's and server endpoints, server_params
is optional and can be used to set option in the :code:`GET` type request.

When the endpoint is successfully called it should return a json type structure containing the path to the item, as an
example using :code:`data_get_map = {'myform': '/api/path/to/formdata'};`:


.. code-block:: json

    {
      "netflow": {
        "capture": {
          "interfaces": {
            "lan": {
              "value": "LAN",
              "selected": 1
            },
            "wan": {
              "value": "WAN",
              "selected": 0
            }
          },
        },
        "collect": {
          "enable": "1"
        }
      }
    }

Which maps to the fields in this simplified structure (usually rendered via our volt templates):

.. code-block:: html

    <form id="myform">
        <select multiple="multiple" id="netflow.capture.interfaces">
        </select>
        <input type="checkbox" id="netflow.collect.enable">
    </form>


The function returns a :code:`$.Deferred()` which will be resolved when all endpoints are called.

----------------------------
saveFormToEndpoint
----------------------------

:code:`saveFormToEndpoint(url, formid, callback_ok, disable_dialog, callback_fail)` is the opposite of :code:`mapDataToFormUI()`
and retrieves the data from the form and sends it to the configured (url) endpoint as json structure.

The response data looks similar to the example data in mapDataToFormUI, but more condensed since selections will
be returned as single (separated) values, such as :code:`lan,wan` if both options where set.
