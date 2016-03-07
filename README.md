A single header modification of the Sundown markdown parser,
which is itself a modification of the earlier Upskirt parser.

As fast, secure, and efficient as the library it's converted from, which
was deprecated in 2012. So take it with a grain of salt.

# INCLUSION
In *ONE* source file, put:

    #define SD_IMPLEMENTATION
    #include "markdown.h"

All others may simply #include "markdown.h"

By default, the HTML renderer is included. To disable it (e.g. if you are not
using it), add above the #include:

    #define SD_NO_HTML

# Compiler warnings

In MSVC v19.x, this header will generate the following two warnings on level 4:

    warning C4100: '____': unreferenced formal parameter
    warning C4146: unary minus operator applied to unsigned type

These will not be fixed for algorithmic/API reasons. I recommend you ignore them.

In GCC, in C99 mode, this header should compile without warnings. It will not
compile in C89 mode.

# DOCUMENTATION

## BASIC USAGE

    struct sd_markdown *md = 
        sd_markdown_new(extensions, max_nesting, &callbacks, &your_data);

    sd_markdown_render(output_buffer, input_data, in_data_size, md);

    sd_markdown_free(md);

### Explanation:

Sundown supports several extensions to the markdown syntax which can be
selectively enabled when creating a new context:

     unsigned int extensions = MKDEXT_TABLE | MKDEXT_SUPERSCRIPT ... ;
  
You will need to decide on a maximum nesting level for the context to use
ahead of time. Most markdown documents don't end up nesting very deep, so
a low number (10-20) is probably fine. 

     size_t max_nesting = 15;

Sundown will call back into your code to render an output document:

     struct sd_callbacks callbacks;

Most of its callbacks are of the form:

     void callback(struct sd_buf *output_buffer, const struct sd_buf *text, void* opaque);

Where `output_buffer` is an `sd_buf` that you write the output string to, and 
text is the raw textual contents of whichever piece of syntax you're getting 
a callback for. The span-level callbacks also need to return a value, which
tells it whether you handled the content specially or if it should just print
it out verbatim. The opaque pointer is some piece of your own data that you 
pass in when you create the markdown parsing context.

     struct sd_markdown *md = 
      sd_markdown_new(extensions, max_nesting, &callbacks, &your_data);

The call to render is when you provide an output buffer. `sd_bufnew` requires an
initial capacity, but you will have the chance to grow the buffer during
rendering callbacks.

     struct sd_buf *output_buffer = sd_bufnew(...);

Input data is simply an array of characters, which for the most part are left 
as-is and thus should more or less support UTF-8. There are a few functions that
make assumptions about character size (for example running tolower() on a single
8-bit character), so watch out for that.

     sd_markdown_render(output_buffer, input_data, in_data_size, md);

And of course, to clean up when you're done:

     sd_markdown_free(md);

## HTML RENDERING

Included is the fairly compliant HTML renderer that was packaged with sundown 
originally. It will be enabled by default unless `SD_NO_HTML` is defined.

     sdhtml_renderer(&callbacks, &options, 0);

     struct sd_markdown *md = 
         sd_markdown_new(extensions, max_nesting, &callbacks, &options);

     sd_markdown_render(output_buffer, input_data, in_data_size, md);

     sd_markdown_free(md);

### Explanation

`sdhtml_renderer` will initialize the struct `sd_callbacks` with the set of html
rendering callbacks defined near the bottom of this file. It will also
initialize a struct `html_renderopt` that holds some information it needs to use
while rendering the document, which you should pass in as the opaque pointer
when creating the markdown parser.

# Philosophy

This port of sundown is crafted in the style of [Sean Barett's `stb_` libraries](
https://github.com/nothings/stb). As this is merely an adaptation of an existing
library, the usual priorities are somewhat diluted, but nonetheless are:

 1. Ease of use
 2. Ease of maintenance
 3. Performance

The first two priorities are reflected in this port in that it is made to be even
more portable than the original library (one header file, as opposed to 8-odd .c
and .h files), uses no dependencies other than the CRT, and strives to be careful
about names in the global namespace. 

# HISTORY

 - v0.90   2016-03-06      First public release


