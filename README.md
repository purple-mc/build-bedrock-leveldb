Automatic builds of Microsoft's fork of [leveldb] for 64-bit Windows (x64 and arm64).

Build produces a static library (.lib file) that is built statically with its dependencies.

Download binary build as zip archive from [latest release] page.

---

Information regarding keys and values stored in the database can be found through the following links:
- [Actor Storage in Minecraft: Bedrock Edition]
- [Bedrock Edition level format]
- [Historic Bedrock Edition level format]

The following is a basic example of how to use the C based api exposed by Microsoft's fork:

```c
#include <stdio.h>
#include <leveldb\c.h>

int main(int argc, char* argv[])
{
        if (argc != 2)
        {
                printf("ERROR: Supply a path to a bedrock db directory.\n");
                exit(-1);
        }
        
        const char* db_path = argv[1];

        // Required for error checking on all database operations.
        char* db_status = NULL;

        // Create config for our database.
        leveldb_options_t* db_options = leveldb_options_create();
        leveldb_options_set_create_if_missing(db_options, 1);

        // Needed to read bedrock world data.
        leveldb_options_set_compression(db_options, leveldb_zlib_raw_compression);

        // Open world.
        leveldb_t* db = leveldb_open(db_options, db_path, &db_status);

        // We need these to read and write to the database.
        leveldb_readoptions_t* read_options = leveldb_readoptions_create();
        leveldb_writeoptions_t* write_options = leveldb_writeoptions_create();

        // Uncomment to use synchronous writes. 
        //leveldb_writeoptions_set_sync(write_options, 1);

        // Create iterator to loop through the entire database.
        leveldb_iterator_t* iter = leveldb_create_iterator(db, read_options);
        for (leveldb_iter_seek_to_first(iter); leveldb_iter_valid(iter); leveldb_iter_next(iter))
        {
                size_t key_size, value_size;
                const char* key = leveldb_iter_key(iter, &key_size);
                const char* value = leveldb_iter_value(iter, &value_size);
                printf("key length: %zu, value length %zu\n", key_size, value_size);
        }

        // Destory the iterator.
        // This must be done before closing a database, otherwise an assertion is thrown.
        leveldb_iter_destroy(iter);

        // Close the database.
        leveldb_close(db);

        return 0;
}
```

[leveldb]: https://github.com/mojang/leveldb
[latest release]: https://github.com/purple-mc/build-bedrock-leveldb/releases/latest
[Actor Storage in Minecraft: Bedrock Edition]: https://learn.microsoft.com/en-us/minecraft/creator/documents/actorstorage?view=minecraft-bedrock-stable
[Bedrock Edition level format]: https://minecraft.wiki/w/Bedrock_Edition_level_format
[Historic Bedrock Edition level format]: https://minecraft.wiki/w/Bedrock_Edition_level_format/History