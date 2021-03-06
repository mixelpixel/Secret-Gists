# AARON posted to Slack, Friday, November 10, 2017
## [1:51 PM (EST)](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510339876000055)
Hope you're all looking forward to a bit more Secret Gists - right now I'm working with the auth part of the node package, as it looks like their documentation is out of date (a sad reality in software development). The good news is instantiating and interacting with most of the API is fine without authentication, you can still either play with pulling down gists or trying to write functions that use tweetnacl.js to encrypt and decrypt strings.

The least out-of-date documentation I've found for the GitHub API is here: https://octokit.github.io/node-github/#api-gists
```
githubCli.gists.getAll({}).then(response => {
  console.log(response);
});
```
That snippet should pull a list of public gists, and you can try logging or parsing them.

If you want to play with tweetnacl.js instead that's also great, I'll put some hopefully helpful/starter snippets for that soon. But basically, you'll need to make `randomBytes` and use those as a key, and make sure to use the `nacl.util` methods for string encoding/decoding.


## [2:00PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510340406000202)
If you do want to play with the authorized GitHub API, I have found that the documented basic (username/password) method does work:
```
githubCli.authenticate({
  type: 'basic',
  username: process.env.USERNAME,
  password: process.env.PASSWORD
});
```
You'd need to set environment variables for USERNAME and PASSWORD to be your GitHub username/password respectively. You also need to set CLIENT_ID and CLIENT_SECRET (what we called GITHUB_TOKEN yesterday) env variables (to the two strings from where you made the app on the GitHub webpage), and you can then get an OAuth token:
```
githubCli.authorization.create({
  client_id: process.env.CLIENT_ID,
  client_secret: process.env.CLIENT_SECRET
}).then(response => {
  console.log(response);
  // save and use response.data.token for the OAuth process
});
```
In principle you really shouldn't need username/password *and* client_id/client_secret, but their functions supporting just the latter aren't returning as documented. As I said, real world API != real world documentation much of the time - but the above does work.

## [3:04PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510344246000315)
One further note on auth - though the client_id/client_secret approach should work, you can directly create an OAuth token here: https://github.com/settings/tokens
This can then be used e.g.:
```
github.authenticate({
  type: 'oauth',
  token: process.env.GITHUB_TOKEN
});
```

## [3:15PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510344947000328)
Once you're authed, you can create gists like this:
```
github.gists.create({
  "description": "the description for this gist",
  "public": true,
  "files": {
    "TEST.md": {
      "content": "<html><h1>This is a Test!</h1><b>Hello</b></html>"
    }
  }
}).then(response => {
  console.log(response);
});
```

## Jake [QUESTION](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345006000030)
So we can authenticate just using
`github.authenticate`
without creating the new github?

## [3:18PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345087000356)
Yeah you can instantiate a more basic GitHub object:
```
const GitHubApi = require('github');
let github = new GitHubApi({
  debug: true
});
```
And then use `github.authenticate` - supposedly all three of these methods work:
```
github.authenticate({
  type: 'oauth',
  token: process.env.GITHUB_TOKEN
});

github.authenticate({
 type: 'basic',
 username: process.env.USERNAME,
 password: process.env.PASSWORD
});

github.authenticate({
  type: 'oauth',
  key: process.env.CLIENT_ID,
  secret: process.env.CLIENT_SECRET
});
```
But the client id/secret method in particular seems aspirational, despite them documenting it.

But if you create a token (https://github.com/settings/tokens) that has permission to create gists, auth with that, then you should be able to do the `github.gist.create` command.

## Ely [QUESTION](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345207000156)
so is this suppose to handle only our logins, or also other persons?

## [3:21PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345318000452)
Just yours for now - an example of handling other people's with a similar tech stack is here: https://stackoverflow.com/questions/24238888/how-to-authenticate-with-github-with-node-and-express
And I'd actually like to get to that, but don't expect to today certainly. It requires having an actual front-end with templates (e.g. React), which you have experience with but would be as lot to tie in right now.

That said, next week I'll be talking about devops, deployment, relational databases, and things like that - so we'll definitely get to these sorts of issues, and later today I'd actually like to chat with you all about interests and what you'd like to see in the upcoming material.

## Jake [QUESTION](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345349000124)
And why do we need to store our username and passwords as environment variables? Is that just a way to hardcode our account information without getting it from a post request?

## [3:23PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345406000103)
Storing your username/password as environment variables is just an alternative to storing the auth token as an environment variable - I'd mostly recommend the latter, since it can have tighter permissions. But yes, it's a way to set your auth info to send it to GitHub via post, and to not have to stick it directly in your code so you can still commit/push.

## Jake [QUESTION 3:28PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510345731000049)
for the authentication example would you use
```github.authenticate```
or
```GitHubApi.authenticate```
?

<details><summary>SUB THREAD</summary><p>

- Aaron Gallant [32 minutes ago]
  - Probably `github.authenticate` - `GitHubApi` is the name assigned to importing the library, while `github` is assigned to the instance that is created with `GitHubApi()`.

- Jake Cooley [29 minutes ago]
  - @Aaron I keep getting
  - ```cannot match against null or undefined```
  - doing this:

  ```
  const github = new GithubApi({
    debug: true
  });

  github.authenticate({
    type: 'oauth',
    key: process.env.CLIENT_ID,
    secret: process.env.CLIENT_SECRET
  });
  ```

- Aaron Gallant [28 minutes ago]
  - Try `let github` instead of `const` - constants are a great practice, but I suspect the github object in this case needs to be able to update itself.

- Jake Cooley [28 minutes ago]
  - let was giving me an error with the linter
  - edit: seems to be working now but with the same result as before (edited)

- Aaron Gallant [26 minutes ago]
  - Working as in the linter accepts let, but you get the same error?

- Jake Cooley [25 minutes ago]
  - It worked when I saved it but when i restarted the server it gave me the linter error again

- Aaron Gallant [25 minutes ago]
  - Ah okay, so the linter error I'm guessing is it doesn't like a top-level `let` - you can try putting it inside a class or function.

- Jake Cooley [22 minutes ago]
  - `error  'github' is never reassigned. Use 'const' instead  prefer-const`
  - That was the specific error

- Aaron Gallant [16 minutes ago]
  - Okay, did some testing and actually `const` seems to work for fine (at least I don't get that error on authenticate). Try starting `node` from the command line in your project path and running:

  ```
  const GitHubApi = require('github');

  const github = new GitHubApi({
    debug: true
  });

  github.authenticate({
    type: 'oauth',
    token: process.env.GITHUB_TOKEN
  });
  ```

- Jake Cooley [15 minutes ago]
  - Is the github token supposed to be the personal access token?

- Aaron Gallant [11 minutes ago]
  - The GitHub token should be made here: https://github.com/settings/tokens
  - And you should check "create gists" for its permissions but nothing else (unless you want to).

- Jake Cooley [7 minutes ago]
  - we juststore it like this?
  - `<cmd> GITHUB_TOKEN=<personal access token>`
  - Not as a string or anything strange like that?

- Aaron Gallant [4 minutes ago]
  - Yeah you can run it directly - you can wrap it on quotes too, e.g. `GITHUB_TOKEN="yourgithubtoken"`. It won't make a difference in this case since the string should just be alphanumeric, but for strings with spaces/funkiness quotes can help on the command line.

- Jake Cooley [1 minute ago]
  - I'm still getting the same error. This is all supposed to be inside the
  - `'/login'`
  - route, correct?

- Aaron Gallant [6 minutes ago]
  - Ah okay, yeah triggering via the route is slightly different. TBH I'd put that route in when I was planning to have an actual other-user login, but then I saw you've not worked with express+front-end yet. I'd put this code just in the top-level of your file and forget about that route for now - it can still be useful, but for the basic logging in just as you you don't need it.

- Jake Cooley [3 minutes ago]
  - Oh, gotcha. How do I test if I'm authenticated then?
  - When I ran those commands through node it let me enter everything without any errors but is there a way to confirm it worked? Or does the lack of errors mean that it worked?

- Aaron Gallant [1 minute ago]
  - The lack of errors probably means it worked, but a way to test would be to run something that will have a side effect, e.g. create a gist:
  ```
  github.gists.create({
    "description": "the description for this gist",
    "public": false,
    "files": {
      "TEST.md": {
        "content": "<html><h1>This is a Test!</h1><b>Hello</b></html>"
      }
    }
  }).then(response => {
    console.log(response);
  });
  ```

  - Then go to gist.github.com as yourself and see if you see it.
  - And you can run that code either manually in node, in the top level of app.js for testing, or actually put it in a route and trigger it by sending a request to express (e.g. a post to /gist or such).

- Jake Cooley [3 minutes ago]
  - I'm guessing it worked. Does this link work for you?  https://gist.github.com/97fb6f58bd1257b0e1dbe5701ac3982e

- Aaron Gallant [< 1 minute ago]
- Yep, it does! You should now also try the `gist.getForUser` call and see if you can get a list that includes that gist.

- Jake Cooley [< 1 minute ago]
  - Would there be an easier way to do this where we throw it all in a get request and res.json everything?

- Aaron Gallant [2 minutes ago]
  - This can definitely be refined - as you noticed, the gist.create call takes JSON, so for actually wiring it up to express you can make a route that listens for a request with a JSON body and passes that on to the GitHub API. As far as res.json, you can also take the success/failure response from GitHub (which it returns as JSON) and give that to express to render as you want. I definitely encourage trying to do both of those things, so you can interact with it more naturally and see the output. (edited)

- Jake Cooley [2 minutes ago]
  - and the gists getAll function seems to work and returns a list of gists. Thanks for the help!

- Aaron Gallant [1 minute ago]
  - gist.getAll works for you? I actually only got gist.getForUser to work, so that's interesting. Glad it's working though, and looking forward to seeing your code!

- Jake Cooley [1 minute ago]
  - Here is what it returned: https://pastebin.com/tu1aHj9F Does that look like it returned the correct information?

- Aaron Gallant [< 1 minute ago]
  - I think what it did is it treated you as unauthenticated, and returned a list of the few most recent public gists across all of GitHub. There are definitely interesting uses for that (some sort of "What's new on GitHub?" app), but yeah if you want to get back *your* gists try the `gist.getForUser({username: "yourusername"}).then(...` call. Also, though it was definitely helpful to see your output, I'd suggest deleting that pastebin as it does have your access_token in it.

- Jake Cooley [2 minutes ago]
  - Whoops, thanks. I deleted the token from my github so it should be fine.
  - Yeah I just authenticated in the global scope and then created a '/' get request where it returns all gists and I'm getting an unauthorized error from postman
  - Maybe I'll try a few other authentication methods and see if I can get anything to work

- Aaron Gallant [< 1 minute ago]
  - K, I also do think authentication should actually work inside the route, but will be looking at this more myself and make sure we have a reference to go over Monday.

</p></details>


## Wesley [QUESTION 3:40PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510346437000238)
I’m trying to understand the REST API v3 page: The owner of the server  registers the application, and creates the server.  When a browser hits the root ‘/’, the server redirects the browser to GitHub’s `login/oauth/authorize` site with a query parameter of `user:email` and includes the `client_id`, which is the registered app’s id.  What does the user enter at GitHub at this point?  The client_secret?  GitHub then redirects the browser to the callback URL, in the example’s case ‘/callback’, and includes a temporary code, that the server then posts back to GitHub, along with both the client_id  and client_secret.  GitHub then sends an access token to the server.  Is this correct?  The document then says:

```
In the future, users will be able to edit the scopes you requested, and your application might be granted less access than you originally asked for.
```

I’m confused who the `user` is in that sentence and who `you` is.  I thought we were in charge of these things in this scenario.

<details><summary>SUB THREAD</summary><p>

- Aaron Gallant [40 minutes ago]
  - This Stack Overflow link may help a bit overall: https://stackoverflow.com/questions/24238888/how-to-authenticate-with-github-with-node-and-express

  - But in general, yeah there is supposed to be a different `you` versus the `user` in this model - the client_id/client_secret are basically `you` (the developer), authorizing the application to interact with GitHub. For the specific `user` then they also need to interact with GitHub to get an OAuth token and use that for further things (e.g. create gists, whatever).

  - In the specific code we're working on this is a little confused because you haven't worked on applications that really combine the back-end and front-end, so `you` and `user` end up being the same, even though in practice it can/should be different.

  - For now I'd suggest getting an OAuth token directly from https://github.com/settings/tokens rather than worrying about the client id/secret, and essentially just being a single user from the server. This can and should be extended to let other users log in, but we won't get to that in this particular app.


- Wesley Harvey [37 minutes ago]
  - Ok; so basically skip everything about getting authenticated, and start with making authenticated requests because the token will be stored in our environment?

- Aaron Gallant [37 minutes ago]
  - Yep - storing the token in the environment is a form of authentication really, but just the shortest and simplest one.

</p></details>


## [3:50PM PST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510347031000229)
One other GitHub API clarification - for getting gists I suggest:
```
github.gists.getForUser({username: "yourusername"}).then(result => console.log(result))
```
`github.gists.getAll()` while authenticated doesn't actually return all the private gists for the user you're authenticated as. As I said earlier, APIs don't always do what they say on the tin... if you dig back into the issues filed for this API you'll see folks complaining about this sort of thing, and it's fairly common. A lot of times reality doesn't live up to design, or somebody changes something with unexpected consequences.

<summary>Thanks all for bearing with the bumps though - I'd assumed the "official" GitHub npm library would be slightly closer to reality than it is, but I do think if you (a) use authenticate with an OAuth token w/gist permissions, and (b) try the gist.create and gist.getForUser examples, it should work.</summary> (edited)

As an aside, in situations where an API library is really poor that can be a reason to go back and implement your own via HTTP requests - I'm not sure this situation is *that* bad, but it can be.

## Thomson Comer [3:56 PM EST](https://lambdaschoolpro.slack.com/archives/G5TDU61DE/p1510347376000469)
Understanding the http API is almost always more useful than understanding a library that is supposed to wrap it. And remember, All software has bugs. :grinning:
