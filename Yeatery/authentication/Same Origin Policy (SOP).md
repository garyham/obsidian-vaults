[go here for a good explaination of SOP](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)

Two URLs have the _same origin_ if the [protocol](https://developer.mozilla.org/en-US/docs/Glossary/Protocol), [port](https://developer.mozilla.org/en-US/docs/Glossary/Port) (if specified), and [host](https://developer.mozilla.org/en-US/docs/Glossary/Host) are the same for both.

| URL                                               | Outcome     | Reason                                           |
|:------------------------------------------------- |:----------- |:------------------------------------------------ |
| `http://store.company.com/dir2/other.html`        | Same origin | Only the path differs |
| `http://store.company.com/dir/inner/another.html` | Same origin | Only the path differs |
| `https://store.company.com/page.html`             | Failure | Different protocol |
| `http://store.company.com:81/dir/page.html`       | Failure | Different port (`http://` is port 80 by default) | 
| `http://news.company.com/dir/page.html`           | Failure | Different host |

A document (html page) or script **cannot** interact with a resource from another origin.

To relax the SOP restrictions we use [[Cross Origin Resource Sharing (CORS)]]