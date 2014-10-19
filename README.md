# MojoAuth

MojoAuth is a set of standard approaches to cross-app authentication based on [HMAC](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code) which is specified in [RFC2104](https://www.ietf.org/rfc/rfc2104.txt).

## Why does MojoAuth exist?

[Mojo Lingo](http://mojolingo.com) regularly implements multi-faceted real time communications applications which bring together a large number of separate, individually complex technologies to produce a useful consumer experience. These kind of projects regularly include situations where an un-trusted client, such as a user's web browser, must communicate directly with more than one back-end system, including SIP, XMPP and TURN servers, as well as the front-end application server, and must do so in an authenticated manner.

These situations present a problem with authentication. How does the un-trusted client prove its identity to many separate systems? In the most simple implementation, a client must submit a username and password through the browser's UI in order to initialize the connections to each separate system, but this is a complex and ugly user experience and requires close coordination of user databases between the backend systems.

In order to create a better user experience, developers then go looking for a way to share an authenticated session between disparate systems. One of the easiest ways of implementing this is to expose some plain-text credentials from the front-end application server to the client, which the client then uses to authenticate with the other back-end systems. Exposing long-lived credentials like this is a security risk, and this assumes that those credentials must somehow be synchronised across all back-end systems, which can be excessively complex.

MojoAuth exists to wrap up an easy-to-remember name, high-level documentation and recommendations, library implementations and industry standards in order to avoid decision overhead and reinventing of the wheel in projects that Mojo Lingo is involved in, and at the same time solving the coordination and security problems inherant in the simple solutions as described above.

MojoAuth is the thing we intend to point our own staff and our clients at when the question of cross-service authentication arises, expecting that it will be used in the majority of cases.

## Did Mojo Lingo invent the technology behind MojoAuth?

No. MojoAuth is based on ideas taken from [draft-uberti-behave-turn-rest](https://tools.ietf.org/html/draft-uberti-behave-turn-rest-00) by Justin Uberti of Google's WebRTC team. All credit for devising and specifying the behaviour of MojoAuth belongs to Justin / Google, and Mojo Lingo are eternally grateful for Justin's contributions to this industry and it's open-source and standards communities.

## How does MojoAuth work?

Authentication flow using MojoAuth goes something like this:

* *Client* establishes session with App Server by unspecified mechanism
* *Client* requests temporary credentials from *App Server*
* *App Server* creates a string which combines the user's identity with the credential expiry time, called the "message"
* *App Server* uses a secret key to cryptopgraphically sign the message, giving the "password", and provides both the message and the password to the *Client*
* *Client* attempts to authenticate with the *third-party service* (which shares the secret key with the *App Server*) using the provided credentials
* The *third-party service* signs the provided message (username) and checks it matches the provided password, confirming that the credentials were generated by the *App Server*. If a match cannot be confirmed, the credentials are rejected
* The *third-party service* splits the message back out to the asserted ID and the expiry time. If the expiry time is in the past, it rejects the credentials, otherwise it accepts authentication for the asserted ID.

This is basically the App Server confirming to the third-party service that it approves, for a limited time, of the client creating a session as the asserted identity without the App Server and the third-party service ever having to communicate. The method prevents loss of long-term credentials to an attacker, does not require server-side coordination other than sharing a secret and does not allow arbitrary ID assertion or extension of credential expiry by a client.

Its downside is that is also does not permit revocation of credentials without invalidating *all* credentials in flight by changing the secret key. This is a feature typically only available to a more complex approach involving [PKI](http://en.wikipedia.org/wiki/Public_key_infrastructure), but it is suggested that the simplicity of this approach outweighs this limitation in a large number of cases.

## How do I implement MojoAuth?

MojoAuth is implemented in handy little libraries in a collection of programming languages. Each of these libraries has its own usage documentation. An example application (a Rails app serving a WebRTC-based calling service backed by Kamailio and rfc5766-turn-server) is [available to demonstrate usage](https://github.com/mojolingo/mojo-auth).

* [Ruby](https://github.com/mojolingo/mojo-auth.rb)
* [Go](https://github.com/mojolingo/mojoauth.go)
* [Erlang](https://github.com/mojolingo/mojoauth.erl)
* [Elixir](https://hex.pm/packages/mojoauth)
* [Node.js](https://www.npmjs.org/package/mojo-auth.js)

### SIP

[Kamailio](http://www.kamailio.org/w/), the popular SIP server, has a MojoAuth compatible authentication module, [auth_ephemeral](http://kamailio.org/docs/modules/4.1.x/modules/auth_ephemeral.html) which is very easy to use.

### TURN

The most popular open-source TURN server implementation, [rfc5766-turn-server](https://code.google.com/p/rfc5766-turn-server/) supports MojoAuth and can be [easily configured to enable it](https://code.google.com/p/rfc5766-turn-server/wiki/turnserver#TURN_REST_API).

### XMPP
