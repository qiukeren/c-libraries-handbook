## Libcurl Programming Manual

### Usage Order

* Initialize libcurl with curl_global_init()
* Get easy interface pointer with curl_easy_init()
* Set options for curl with curl_easy_setopt()
* Perform curl with curl_easy_perform()
* Clean memory with curl_easy_cleanup()
* Clean global with curl_global_cleanup()


#### curl_global_init(long flags)

Initialize some of the libcurl functionality globally *once* for your program's entire life time.

flags:

* CURL_GLOBAL_ALL
* CURL_GLOBAL_SSL
* CURL_GLOBAL_WIN32
* CURL_GLOBAL_NOTHING

curl_easy_perform() detects if curl_global_init() hasn't been executed and do curl_global_init()'s default options.

Repeated calls to curl_global_init() should be avoided. 

#### curl_easy_init()

Create an easy handle for each session. Do not share the same handle in multiple threads.
Usage:`easyhandle=curl_easy_init()`

#### curl_easy_setopt()

Set properties for a handle.
One of the most basic properties is to set the url: `curl_easy_setopt(easyhandle, CURLOPT_URL, "http://domain.com/");`

Flags:

* CURLOPT_URL
* CURLOPT_WRITEFUNCTION,CURLOPT_WRITEDATA
* CURLOPT_HEADERFUNCTION,CURLOPT_HEADERDATA
* CURLOPT_READFUNCTION,CURLOPT_READDATA
* CURLOPT_NOPROGRESS,CURLOPT_PROGRESSFUNCTION,CURLOPT_PROGRESSDATA
* CURLOPT_TIMEOUT,CURLOPT_CONNECTIONTIMEOUT
* CURLOPT_FOLLOWLOCATION


Important flags:
CURLOPT_TIMEOUT is to set maximum time the request is allowed to take and CURLOPT_CONNECTIONTIMEOUT is to set timeout for the connect phase.
CURLOPT_WRITEFUNCTION is a callback for writing data.
CURLOPT_WRITEDATA is a data pointer to pass to the write callback.By default, this is a FILE * to stdout.

#### curl_easy_perform()

Connect to the remote site, do the necessary commands and receive the transfer.

Usage:`success=curl_easy_perform(easyhandle)`

#### curl_global_cleanup()

When the program no longer uses libcurl, it should call curl_global_cleanup().
Repeated calls to curl_global_cleanup() should be avoided. 


### Sample

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>

struct string {
  char *ptr;
  size_t len;
};

void init_string(struct string *s) {
  s->len = 0;
  s->ptr = malloc(s->len+1);
  if (s->ptr == NULL) {
    fprintf(stderr, "malloc() failed\n");
    exit(EXIT_FAILURE);
  }
  s->ptr[0] = '\0';
}

size_t writefunc(void *ptr, size_t size, size_t nmemb, struct string *s)
{
  size_t new_len = s->len + size*nmemb;
  s->ptr = realloc(s->ptr, new_len+1);
  if (s->ptr == NULL) {
    fprintf(stderr, "realloc() failed\n");
    exit(EXIT_FAILURE);
  }
  memcpy(s->ptr+s->len, ptr, size*nmemb);
  s->ptr[new_len] = '\0';
  s->len = new_len;

  return size*nmemb;
}

int main(void)
{
  CURL *curl;
  CURLcode res;

  curl = curl_easy_init();
  if(curl) {
    struct string s;
    init_string(&s);

    curl_easy_setopt(curl, CURLOPT_URL, "curl.haxx.se");
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, writefunc);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &s);
    res = curl_easy_perform(curl);

    printf("%s\n", s.ptr);
    free(s.ptr);

    /* always cleanup */
    curl_easy_cleanup(curl);
  }
  return 0;
}

```
