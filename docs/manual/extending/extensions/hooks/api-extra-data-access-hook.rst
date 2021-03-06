.. _api-extra-data-access-hook:

======================
APIExtraDataAccessHook
======================


.. currentmodule:: reviewboard.webapi.base

:py:class:`reviewboard.extensions.hooks.APIExtraDataAccessHook` is used
when an extension wants to give an ``extra_data`` field certain accessibility
rules by registering an access state to a given key. The access states are:

    :py:data:`ExtraDataAccessLevel.ACCESS_STATE_PUBLIC`
        The extra data key can be retrieved and updated via the API.

    :py:data:`ExtraDataAccessLevel.ACCESS_STATE_PUBLIC_READONLY`
        The extra data key can only be retrieved via the API. It cannot be
        updated.

    :py:data:`ExtraDataAccessLevel.ACCESS_STATE_PRIVATE`
        The extra data key can neither be retrieved nor updated via the API.

Extensions wishing to use this functionality must provide:

 1. the :py:class:`resource <WebAPIResource>` they wish to hook into, and
 2. a mapping of each path (as a tuple of keys) to the desired access state.

For example, consider the following ``extra_data`` object:

.. code-block:: python

   extra_data = {
       'pgp': {
           'public_key': 'my public key',
           'private_key': 'my private key',
       },
   }

In this case, to make the ``private_key`` field private, the field set would
be:

.. code-block:: python

   {
       ('pgp', 'private_key'): ExtraDataAccessLevel.ACCESS_STATE_PRIVATE,
   }


The :py:meth:`~reviewboard.extnesions.hooks.APIExtraDataAccessHook.get_extra_data_state`
method can also be overridden to provide more complex behaviour in determining
the access level of an extra data key. An example follows:

Example
=======

.. code-block:: python

   from reviewboard.extensions.base import Extension
   from reviewboard.extensions.hooks import APIExtraDataAccessHook
   from reviewboard.webapi.base import WebAPIResource, ExtraDataAccessLevel
   from reviewboard.webapi.resources.review_request import \
       ReviewRequestResource


   class CustomAccessHook(APIExtraDataAccessHook):
       """An extra data access hook that defaults to private.

       All extra_data keys on associated resources will be marked as private
       (and therefore not returned by the API) unless specified otherwise
       """

       def get_extra_data_state(self, key_path):
           for path, access_state in self.field_set:
               if path == key_path:
                   return access_state

           return ExtraDataAccessLevel.ACCESS_STATE_PRIVATE

   class SampleExtension(Extension):
       def initialize(self):
           review_request_resource = ReviewRequestResource()

           review_request_resource.extra_data = {
               'public': 'foo',
               'private': 'secret',
               'readonly': 'bar',
           }

           access_hook = CustomAccessHook(
               self,
               review_request_resource,
               {
                   ('public',): ExtraDataAccessLevel.ACCESS_STATE_PUBLIC,
                   ('readonly',): ExtraDataAccessLevel.ACCESS_STATE_PUBLIC_READONLY,
               })
