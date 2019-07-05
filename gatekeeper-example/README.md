# gatekeeper-example
Below is an example Docker compose file to protect the unauthenticated hello-world application with Keycloak GateKeeper.  
So the flow is as follows:
  - Your HTTP request to `hello-world.example.org` hits the GateKeeper proxy.
  - The proxy checks your browser for the encrypted token.
  - If its not present it will redirect you to the Keycloak login page.
  - You login with your user account.
  - Keycloak will redirect you back to `hello-world.example.org`
  - If you have the correct roles, GateKeeper will proxy you to the hello-world app.

```
version: "3"

services:
  hello-world:
    image: tutum/hello-world
    container_name: hello-world
    restart: always
    expose:
      - 80

  hello-world-proxy:
    container_name: hello-world-proxy
    image: keycloak/keycloak-gatekeeper
    restart: always
    command: >
      --discovery-url=https://your-keycloak-domain.org/auth/realms/your-realm-name
      --upstream-url=http://hello-world
      --redirection-url=http://hello-world.example.org
      --client-id=hello-world
      --client-secret=f3d6e571-3c88-dbac-a151-83b791a34b99
      --encryption-key='e36Rh8v4UFmCy7wUxE23zpFPKcrBsJcu'
      --cookie-domain=example.org
      --listen=0.0.0.0:3000
      --enable-refresh-tokens=true
      --enable-encrypted-token=true
      --enable-logout-redirect=true
      --forbidden-page=/forbidden.html.tmpl
      --enable-token-header=false
      --enable-authorization-header=false
      --resources="uri=/*|roles=hello-world-access"
    volumes:
      - ./forbidden.html.tmpl:/forbidden.html.tmpl
    ports:
      - 80:3000

```

### Gatekeeper Global Options
Below is the global options I have chosen for my standard setup, with some info that explains what it is.
A list of all options can be found in the `gatekeeper-global-options.txt` file.
```
--discovery-url=https://your-keycloak-domain.org/auth/realms/your-realm-name
  # Discovery url to retrieve the openid configuration
  # [URL to your Keycloak realm]

--upstream-url=http://hello-world
  # URL for the upstream endpoint you wish to proxy
  # [The URL til the application behind the GateKeeper]

--redirection-url=http://the-url-to-this-application.example.org 
  # Redirection url for the oauth callback url, defaults to host header is absent
  # [The global URL to the application]

--client-id=hello-world
  # Client id used to authenticate to the oauth service

--client-secret=f3d6e571-3c88-dbac-a151-83b791a34b99
  # Client secret used to authenticate to the oauth service

--encryption-key='e36Rh8v4UFmCy7wUxE23zpFPKcrBsJcu'
  # Encryption key used to encryption the session state
  # [Random characters, just create your own.]

--cookie-domain=example.org
  # Domain the access cookie is available to, defaults host header

--listen=0.0.0.0:3000
  # The interface the service should be listening on
  # [0.0.0.0 means "any interface"]
  
--enable-refresh-tokens=true
  # Enables the handling of the refresh tokens (default: false)

--enable-encrypted-token=true
  # Enable encryption for the access tokens (default: false)

--enable-logout-redirect=true
  # Indicates we should redirect to the identity provider for logging out (default: false)

--forbidden-page=/forbidden.html.tmpl 
  # Path to custom template used for access forbidden
  # [This line is optional and will present a more stylish "Access denied" page.]

--enable-token-header=false
  # Enables the token authentication header X-Auth-Token to upstream (default: true)

--enable-authorization-header=false
  # Adds the authorization header to the proxy request (default: true)

--resources="uri=/*|roles=hello-world-access"
  # List of resources 'uri=/admin*|methods=GET,PUT|roles=role1,role2'
  # [The roles needed to gain access to the application behind GateKeeper]
```
