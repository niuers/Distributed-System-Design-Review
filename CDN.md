# CDN

1. CDN can be very important to improve the performance by pushing the popular videos (youtube) to CDN, and reduce latency.
   * Flash crowd problem for popular websites, request load overwhelms some aspect of the site’s infrastructure, such as the frontend Web server, network equipment, or bandwidth, or (in more advanced sites) the back-end transaction-processing infrastructure.
   * By caching content at the Internet’s edge, we reduce demand on the site’s infrastructure and provide faster service for users, whose content comes from nearby servers.
   * 

1. A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user. Generally, static files such as HTML/CSS/JS, photos, and videos are served from CDN, although some CDNs such as Amazon's CloudFront support dynamic content. The site's DNS resolution will tell clients which server to contact.

1. Serving content from CDNs can significantly improve performance in two ways:
   * Users receive content from data centers close to them
   * Your servers do not have to serve requests that the CDN fulfills



## Types of CDN
### Push CDNs
1. Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.
1. Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

### Pull CDNs

1. Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.
1. A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.
1. Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

## Disadvantage(s): CDN
1. CDN costs could be significant depending on traffic, although this should be weighed with additional costs you would incur not using a CDN.
1. Content might be stale if it is updated before the TTL expires it.
1. CDNs require changing URLs for static content to point to the CDN.

