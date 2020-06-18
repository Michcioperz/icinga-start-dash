# icinga-start-dash

This is a very basic service grid for replacing Icingaweb with something that doesn't leak memory (could be just that our Icinga is outdated but I'm leaving this job this month anyway).

## Get started

```bash
npm install
npm run build
```

And then slap the resulting contents of `public` somewhere on the same domain as your Icingaweb I guess? I mean, this way the authorization is shared without you having to mess with CORS.
