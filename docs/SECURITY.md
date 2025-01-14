# SECURITY

This pages focuses on providing information on making the platform deployed with alfresco-ansible-deployment secure.
In particular, for Share to work, Follow the [Share security setup](#Share-security-setup).

## Specify trustworthy applications

Nowadays several security features rely on the fact the server tells the browser which are the applications that can be trusted.
This is largely due to the expanding usage of single pages applications and setups where such an application is hosted on a domain name which do not match the backend server.

In such circumstances, you can tell the playbook which are these applications by using adding the client application to your inventory file as shown below:

```yaml
all:
  chidren:
    other_repo_clients:
      hosts:
        my_custom_app:
          known_urls:
            - http://app.domain.local/legit
            - https://app.domain.local/legit
```

Note the `known_urls` variable. It needs to be defined as a list of URLs where the client application is hosted.

## Configure CSRF

CSRF is enabled by default and its default configuration is to only allow requests from the same `Origin` & `Referer` headers. Any other `Origin` or `Referer` will be denied.
If you want a third party application (often SPA) to be able query the alfresco API you need to add it to the list of `known_urls` as shown above.
In addition you can also control whether you want to enforce presence of either or both of the `Referer` or `Origin` headers. This is turned off by default as some
browser/clients simply do not provide them. You should not enforce them unless you have full control on your clients' browser fleet and know they all provide necessary
headers in all circumstances. If so, you can override the default `csrf` configuration node.

```yaml
csrf:
  enabled: true
  force_headers:
    - referer
    - origin
  urls: "{{ trusted_urls }}"
```

Similarly in order to disable CSRF completely use:

```yaml
csrf:
  enabled: false
```

## Configure CORS

The playbook now enables CORS headers to be sent by the server by default. The default policy basically only allows the Same-Origin policy.
If you want to allow for more origins to query  the Alfresco repository, you should make sure they are part of the `known_urls` variable.
Any location mentioned in this variable will be automatically to the list of relaxed origins for CORS queries.
By default they will be allowed tu use any following methods: DELETE, GET, HEAD, OPTIONS, POST, PUT. If you want to restrict methods or further
tweak the CORS configuration, you can do so by overriding the whole `cors` variable as shown bellow:

```yaml
cors:
  enabled: true
  urls: "{{ trusted_urls }}"
  allowed_methods:
    - GET
    - OPTIONS
    - POST
    - PUT
  allowed_headers:
    - Accept
    - Access-Control-Request-Headers
    - Access-Control-Request-Method
    - Authorization
    - Cache-Control
    - Content-Type
    - Origin
    - X-CSRF-Token
    - X-Requested-With
  exposed_headers:
    - Access-Control-Allow-Origin
    - Access-Control-Allow-Credentials
  support.credentials: true
  preflight_maxage: 10
```

To completely disable CORS simply use:

```yaml
cors:
  enabled: false
```

## Share security setup

Share is **always** deployed together with the repo (it's part of the same role), and as a consequence will always try to access the repo through the `localhost` interface.
That means from the repo's point of view - unless Share itself is accessed using [http://localhost/share/](http://localhost/share/) - it is breaking CORS protection.
For that reason in order for Share to work, it is mandatory to add the URL Share will be accessed from as `known_urls`. One way to do it is to add this variable to the
`repository` group as shown bellow:

```yaml
all:
  chidren:
    repository:
      vars:
        known_urls:
          - https://ecm.domain.local/share
      hosts:
        repository_1:
```

## Transformations security

Renditions using the LibreOffice transformer to render HTML are subject to the BSSRF attacks. In order to mitigate this risk, you may decide to disable the optimal renditions
and instead generate lower quality renditions (where basically images would not be rendered as part of the HTML).
That is achieved by the variable bellow:

```yaml
bssrf_protection_enabled: true
```

> This protection is disabled by default so users benefit from the best quality renditions.
