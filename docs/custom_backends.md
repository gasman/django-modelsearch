(modelsearch_custom_backends)=

# Custom backends

```{eval-rst}
Django Modelsearch backends implement the interface defined by :py:class:`modelsearch.backends.base.BaseSearchBackend`. To support the standard search functionality, the backend's ``search()`` method must return a collection of objects or ``model.objects.none()``. For a fully-featured search backend, examine the Elasticsearch backend code in ``elasticsearchbase.py``.
```

## Indexing for alternative search interfaces

Django Modelsearch provides a set of signal handlers and the `rebuild_modelsearch_index` management command to allow backends to keep their own data store in sync with the real model data. Sometimes, it may be desirable to repurpose this mechanism to manage a data store that does not align with Modelsearch's concept of querying - for example, a natural language search that responds to queries with human-readable text rather than a list of results.

This can be achieved by implementing a backend that provides `index_class` and `rebuilder_class` but not the other helper classes such as `query_compiler_class`. For example:

```python
# natural_language/backend.py
from modelsearch.backends.base import BaseIndex, BaseSearchBackend


class NaturalLanguageSearchIndex(BaseIndex):
    def add_model(self, model):
        # Configure the index to accept instances of class `model`

    def add_items(self, model, items):
        # Add a list of items of class `model` to the index
        pass


class NaturalLanguageSearchRebuilder:
    def __init__(self, index):
        self.index = index

    def start(self):
        # Perform any initialization required to start the rebuild process
        return self.index

    def finish(self):
        # Perform any finalization required to finish the rebuild process
        pass


class NaturalLanguageSearchBackend(BaseSearchBackend):
    index_class = NaturalLanguageSearchIndex
    rebuilder_class = NaturalLanguageSearchRebuilder

    def natural_language_search(self, query):
        return "your response here"


SearchBackend = NaturalLanguageSearchBackend
```

This can then be configured as a secondary search backend, to ensure that the index will be kept updated by the signal handlers and `rebuild_modelsearch_index` management command:

```python
MODELSEARCH_BACKENDS = {
    "default": {
        "BACKEND": "modelsearch.backends.database",
        "INDEX": "myproject",
    },
    "natural_language": {
        "BACKEND": "natural_language.backend",
    },
}
```

```{eval-rst}
Since this backend does not implement the regular ``search`` method, it cannot be used as the default backend. However, it can still be retrieved through the :py:func:`modelsearch.backends.get_search_backend` function:
```

```pycon
>>> from modelsearch.backends import get_search_backend
>>> backend = get_search_backend("natural_language")
>>> backend.natural_language_search("What is the airspeed velocity of an unladen swallow?")
"your response here"
```
