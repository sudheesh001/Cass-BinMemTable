### Architecture

When adding a new key to an SSTable here are the steps it goes through. All keys are sorted before writing.

1. Serialize Index (ColumnIndexer.serialize(!IIterableColumns columns, DataOutput dos))
	Sort columns for key
	Serialize columns bloom filter
Loop through columns and subcolumns that make up for column family
Build sum for columnCount by column getObjectCount (includes getting subcolumn counts for super columns)
Create bloom filter with column count
Loop through columns (again) and add column name to bloom filter
If super column detected, loop through subcolumns and add column name
Write bloom filter hash count (int)
Write serialized bloom filter length (int)
Write serialized bytes of bloom filter
Start indexing based on column family comparator
If columns empty write integer zero, return
Iterator over all columns creating a collection of IndexHelper.IndexInfo objects each IndexInfo representing at most getColumnIndexSize() worth of data (default is 64KB: Value from yaml's column_index_size_in_kb)
Construct each new IndexInfo that consists of first and last columns visited that fit in the index size limit
Write size of indexSizeInBytes (int)
Serialize each IndexInfo object - (firstname is first column name visited in block, and lastname is the last column name visited)
Write byte firstname - (length >> 8) & 0xFF
Write byte firstname - (length & 0xFF)
Write byte firstname
Write byte lastname - (length >> 8) & 0xFF
Write byte lastname - (length & 0xFF)
Write byte lastname
Write long startPosition
Write long endPosition - startPosition
Serialize Data (ColumnFamilySerializer.serializeForSSTable(ColumnFamily columnFamily, DataOutput dos)
Write columnFamily localDeletionTime (int)
Write columnFamily markedForDeleteAt (long)
Sort columns
Write the number of columns (int)
Determine Column Serializer and Serialize Column
Determine length of column name as length
Write byte - (length >> 8) & 0xFF
Write byte - length & 0xFF
Write byte of column name
Write boolean isMarkedForDelete
Write long timestamp
Write column value length (int)
Write column value as byte
Write to SSTable Data File
Write out row key in UTF, this is based on partitioner
Random Partitioner
key token + DELIMITER + key name
Delimiter is colon
Write size of row value (int)
Write byte of row value
Write SSTable Bloom Filter and SSTable Index
Add to bloom filter disk key based on partitioner
Random Partitioner
key token + DELIMITER + key name
Delimiter is colon
Write disk key to SSTable Index file (UTF)
Write file position before (Write to SSTable Data File) (int)