# This is an unofficial version of rmongodb --
## I have not yet changed the DESCRIPTION file!
### It does not yet correctly compile rmongodb.so, so it will fail install.
#### Sometime in the summer of 2016 rmongodb was archived at the request of its maintainer.
#### As a package it was still completely functional, but the version of the c driver it shipped with is not supported by the latest version of MongoDB (any version greater than 2.6 requires some special authentication. I haven't gotten that far yet...
#### I was hoping to upgrade the package with the latest stable c driver, 1.4.1. It is slow going; at some point, probably around version 1.0, the libmongoc driver and libbson driver changed the nomenclature of all variables. Here's what I've done so far:

**Notes:**  
configure both libmongoc and libbson  

    ./configure --enable-experimental-features=yes --with-libbson=bundled  --enable-ssl=no --enable-sasl=no  
    ? --enable-ssl=[auto/no/openssl/darwin]: prob need to find a way to do this in R for SCRAM-SHA-1 authentication.  
    ? --enable-sasl=[auto/yes/no] same as above  
convert many \<global headers\> to "local headers":  
Change include \<bson.h\> to include "bson.h"  

    perl -pi -w -e 's/#include <bson.h>/#include "bson.h"/g;' $( grep -rl '#include <bson.h>' )  

Change include <bcon.h> to include "bcon.h"  

    perl -pi -w -e 's/#include <bcon.h>/#include "bcon.h"/g;' $( grep -rl '#include <bcon.h>' )  

Change include <bson-string.h> to include "bson-string.h"  

    perl -pi -w -e 's/#include <bson-string.h>/#include "bson-string.h"/g;' $( grep -rl '#include <bson-string.h>' )  

move src to same dir  
remove refs to yajl dir  
perl -pi -w -e 's/<yajl\//</g;' $( grep -rl '<yajl/' )  
change yajl includes to local files in:  

    mongoc-bson-yajl/bson-json.c:#include <yajl/yajl_parser.h>  
    mongoc-bson-yajl/bson-json.c:#include <yajl/yajl_bytestack.h>  
    mongoc-bson-yajl/yajl_version.c:#include <yajl/yajl_version.h>  
    mongoc-bson-yajl/yajl_gen.h:#include <yajl/yajl_common.h>  
    mongoc-bson-yajl/yajl_version.h:#include <yajl/yajl_common.h>  
    mongoc-bson-yajl/yajl_tree.h:#include <yajl/yajl_common.h>  
    mongoc-bson-yajl/yajl_parse.h:#include <yajl/yajl_common.h>  
    
using:  

    perl -pi -w -e 's/#include <yajl_parser.h>/#include "yajl_parser.h"/g;' $( grep -rl '#include <yajl_parser.h>' )  
    perl -pi -w -e 's/#include <yajl_bytestack.h>/#include "yajl_bytestack.h"/g;' $( grep -rl '#include <yajl_bytestack.h>' )  
    perl -pi -w -e 's/#include <yajl_version.h>/#include "yajl_version.h"/g;' $( grep -rl '#include <yajl_version.h>' )  
    perl -pi -w -e 's/#include <yajl_common.h>/#include "yajl_common.h"/g;' $( grep -rl '#include <yajl_common.h>' )  




add vars -DMONGOC_COMPILATION -DBSON_COMPILATION to Makevars  

change variables mongo -> mongoc_client_t  
from mongo-client.h:

```
/**
 * mongoc_client_t:
 *
 * The mongoc_client_t structure maintains information about a connection to
 * a MongoDB server.
*/
```



change variable bson -> bson_t  

from bson-types.h:  

```
/**
 * bson_t:
 *
 * This structure manages a buffer whose contents are a properly formatted
 * BSON document. You may perform various transforms on the BSON documents.
 * Additionally, it can be iterated over using bson_iter_t.
 *
 * See bson_iter_init() for iterating the contents of a bson_t.
 *
 * When building a bson_t structure using the various append functions,
 * memory allocations may occur. That is performed using power of two
 * allocations and realloc().
 *
 * See http://bsonspec.org for the BSON document spec.
 *
 * This structure is meant to fit in two sequential 64-byte cachelines.
*/
```

(I've started this in utility.h, api_bson.h)   



R CMD build rmongodb_0.1.1 --no-build-vignettes  

install to test compile:  
R CMD INSTALL rmongodb_1.8.0.tar.gz  

or just build shared lib:  
R CMD SHLIB -o rmongodb.so a.f b.f -L/\<opt/acml3.5.0/gnu64/lib\> -l\<acml\>  


view contents of dynamic shared object:  
nm -D --defined-only rmongodb.so  



