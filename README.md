# mrd-storage-client
A Python client for the [MRD Storage Server](https://github.com/ismrmrd/mrd-storage-server)

# Quickstart
## Starting the MRD Storage Server
Start an MRD Storage Server instance using Docker:
```
docker run -p 3333:3333 ghcr.io/ismrmrd/mrd-storage-server:latest
```

## Basic Usage
Store and retrieve a simple dictionary:
```
from mrd_storage_client import Storage

# Initialize connection to the storage server
storage = Storage("localhost", 3333)

# Store a dictionary
example_dict = {"key": "value"}
storage.store(example_dict)

# Retrieve the latest stored object
fetched_dict = storage.fetch_latest()
assert example_dict == fetched_dict

# Close the connection
storage.close()
```

# Usage Examples
## Storing Multiple Objects with Metadata
Store a series of objects with custom names, TTL, and tags:
```
import numpy as np
from mrd_storage_client import Storage

# Initialize connection to the storage server
storage = Storage("localhost", 3333)

# Store multiple arrays with the same name but different tags
for i in range(3):
    # Create a random array
    my_array = np.random.rand(32, 32)
    
    # Store with metadata
    storage.store(
        my_array,
        name="test_data",  # Same name for all arrays
        ttl="1h",          # Time-to-live: 1 hour
        custom_tags={
            "test_id": f"test{i+1:03d}",
            "version": "1.0"
        }
    )
    
    print(f"Stored array with test_id test{i+1:03d}")

# Close the connection
storage.close()
```
## Retrieving Multiple Objects
Fetch multiple objects matching specific criteria:

```
from mrd_storage_client import Storage

storage = Storage("localhost", 3333)

# Fetch all objects with name "test_data"
results = storage.fetch(name="test_data")
print(f"Retrieved {len(results)} arrays")

for i, result in enumerate(results):
    print(f"Array {i+1} shape: {result.shape}")

# Fetch only objects with a specific tag
filtered_results = storage.fetch(
    name="test_data", 
    custom_tags={"test_id": "test003"}
)
print(f"Retrieved {len(filtered_results)} arrays with test_id=test003")

# Close the connection
storage.close()
```


## Working with Blobs Directly
Iterate through blob objects:
```
from mrd_storage_client import Storage

storage = Storage("localhost", 3333)

# Fetch blobs as an iterator
for blob in storage.fetch_blobs(name="test_data"):
    # Access blob metadata
    test_id = blob.get("test_id")
    version = blob.get("version")
    expires = blob.get("expires")
    
    # Get the actual data from the blob
    data = blob.get_object()
    
    print(f"Processing experiment {test_id} (v{version})")
    print(f"Expires: {expires}")
    print(f"Data shape: {data.shape}")
    
# Close the connection
storage.close()
```

# API Reference
For a complete API documentation, see the [mrd-storage-server](https://github.com/ismrmrd/mrd-storage-server) repo.
