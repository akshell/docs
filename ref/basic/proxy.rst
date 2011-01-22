=====
Proxy
=====

.. class:: Proxy(handler)

   A ``Proxy`` object intercepts property access on it. The *handler*
   object must have five attributes::

      new Proxy(
        {
          get: function (name) {
            // Return the property value or undefined if not found
          },

          set: function (name, value) {
            // Set the property value
          },

          del: function (name) {
            // Delete the property and return true;
            // if the property cannot be deleted, return false
          },

          query: function (name) {
            // Return true if the proxy has the property;
            // otherwise return false
          },

          list: function () {
            // Return an Array of all property names
          }
        })
