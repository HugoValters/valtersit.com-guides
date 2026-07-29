> 📖 **Original article:** [HestiaCP FastCGI Caching Secrets for PHP Apps](https://www.valtersit.com/guides/hosting/hestiacp-fastcgi-caching-secrets-for-php-apps/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Agency site goes down under 100 concurrent users. CPU pegged at 90%, but `htop` shows most cores idle – just waiting on PHP-FPM. You’re about to throw money at a bigger VPS. Stop.

I’ve seen this play out a hundred times. The default HestiaCP stack is pristine but ships with zero caching. Every request to a PHP app spawns a full process, warms opcache, hits the database, and renders HTML – only to be thrown away. That single WordPress install on a 1GB VPS could handle 500 concurrent users with **15 lines of Nginx config**.

This guide is for HestiaCP admins who want to squeeze every bit of performance without buying hardware. By the end, you’ll have a production‑grade FastCGI cache with automated purging, security hardening, and monitoring – plus the war stories to convince your boss you don’t need a bigger server.

:::note[TL;DR]
- Add `fastcgi_cache_path` and `fastcgi_cache_key` to your HestiaCP domain config – 15 lines transform performance.
- Always include `$host`, `$request_uri`, and `$request_method` in the cache key. Never cache authenticated content without segmentation.
- Implement a PURGE location block restricted to internal IPs and wire it to your CMS’s save hooks.
- Monitor `X-Cache-Status` headers: if HIT ratio 1M files) |
| `5`    | /a/b/c/d/e/f...| ~16                   | Very high       | Overkill for VPS |

I always start with `levels=1:2`. You’d need 10 million cache files for that to become a problem – not typical for a HestiaCP server.

### Avoiding Cache Stampede with Cache Locks and Stale While Revalidate

When multiple concurrent requests arrive for the same uncached page (e.g., after a cache flush), all of them hit PHP‑FPM, potentially crashing your server. Cache locks prevent that:

```nginx
fastcgi_cache_lock on;
fastcgi_cache_lock_age 5s;
```
- One request populates the cache; others wait for up to 5 seconds. If it takes longer, they fall through to PHP.

Additionally, serve stale content while revalidating:

```nginx
fastcgi_cache_use_stale error timeout updating invalid_header http_500 http_502 http_503 http_504;
```

This means: if the cache has a stale (expired) entry, and the backend is broken or slow, serve the stale version. Your users see *something* instead of error pages.

**Opinion:** Many admins neglect `cache_lock` and then wonder why their database crashes during a flash sale. It’s a hidden killer.

## Cache Purging Strategies – Automating Invalidation

### Why Purging Is Critical

Static caching works great until a comment is added or a product stock changes. Without purging, users see stale data. You need to delete the specific cache entry for updated URLs.

### Manual Purging: curl PURGE Request – Nginx Location Block

Add this inside your `server` block (outside the PHP location):

```nginx
location ~ /purge(/.*) {
    allow 127.0.0.1;
    allow 192.168.1.0/24;
    deny all;
    fastcgi_cache_purge $scheme$host$1;
}
```

Test it:
```bash
curl -X PURGE http://yoursite.com/some-page/
```

Verify with `X-Cache-Status: PURGE` in response headers.

**Security note:** Always restrict PURGE to internal IPs. I’ve seen setups with public PURGE – instant DDoS into origin.

### Automated Purging via PHP Scripts

In WordPress, hook into the `save_post` action:

```php
function my_purge_on_save($post_id) {
    $url = get_permalink($post_id);
    wp_remote_request($url, ['method' => 'PURGE']);
}
add_action('save_post', 'my_purge_on_save');
```

For Laravel, create a job that purges after model updates:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/hosting/hestiacp-fastcgi-caching-secrets-for-php-apps/](https://www.valtersit.com/guides/hosting/hestiacp-fastcgi-caching-secrets-for-php-apps/)**
