# Reduce Realm File Size - C++ SDK
Over time, the storage space used by Realm might become fragmented
and take up more space than necessary. To rearrange the internal storage and
potentially reduce the file size, the realm file needs to be compacted.

Realm's default behavior is to automatically compact a realm file
to prevent it from growing too large. You can use manual compaction strategies when
automatic compaction is not sufficient for your use case.

## Automatic Compaction
The SDK automatically compacts Realm files in the background by continuously reallocating data
within the file and removing unused file space. Automatic compaction is sufficient for minimizing the Realm file size
for most applications.

Automatic compaction begins when the size of unused space in the file is more than twice the size of user
data in the file. Automatic compaction only takes place when
the file is not being accessed.

## Manual Compaction Options
Manual compaction can be used for applications that require stricter
management of file size.

Realm manual compaction works by:

1. Reading the entire contents of the realm file
2. Writing the contents to a new file at a different location
3. Replacing the original file

If the file contains a lot of data, this can be an expensive operation.

Use the
should_compact_on_launch()
method on the database configuration to attempt to compact the database.
Specify conditions to execute this method, such as:

- The size of the file on disk
- How much free space the file contains

The following example shows setting the conditions to compact a realm if the
file is above 100 MB and 50% or less of the space in the realm file is used.

```cpp
// Create a database configuration.
auto config = realm::db_config();

config.should_compact_on_launch([&](uint64_t totalBytes, uint64_t usedBytes) {
  // totalBytes refers to the size of the file on disk in bytes (data + free
  // space). usedBytes refers to the number of bytes used by data in the file
  // Compact if the file is over 100MB in size and less than 50% 'used'
  auto oneHundredMB = 100 * 1024 * 1024;
  return (totalBytes > oneHundredMB) && (usedBytes / totalBytes) < 0.5;
});

// The database is compacted on the first open if the configuration block
// conditions were met.
auto realm = realm::db(config);

```

### Tips for Manually Compacting a Realm
Manually compacting a realm can be a resource-intensive operation.
Your application should not compact every time you open
a realm. Instead, try to optimize compacting so your application does
it just often enough to prevent the file size from growing too large.
If your application runs in a resource-constrained environment,
you may want to compact when you reach a certain file size or when the
file size negatively impacts performance.

These recommendations can help you start optimizing compaction for your
application:

- Set the max file size to a multiple of your average realm state
size. If your average realm state size is 10MB, you might set the max
file size to 20MB or 40MB, depending on expected usage and device
constraints.
- As a starting point, compact realms when more than 50% of the realm file
size is no longer in use. Divide the currently used bytes by the total
file size to determine the percentage of space that is currently used.
Then, check for that to be less than 50%. This means that greater than
50% of your realm file size is unused space, and it is a good time to
compact. After experimentation, you may find a different percentage
works best for your application.
