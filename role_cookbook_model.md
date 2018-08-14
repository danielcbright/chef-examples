# Role Cookbook Model

This is intended to show a working example of the "Role Cookbook Model". By 
following the steps outlined below, you can setup a Chef Environment to use this
model and try it out for yourself. Most of this follows the model set forth by
Jeremy Miller @jeremymv2, a colleague of mine @Chef, you can see his original 
Environment Pinning guide here - https://github.com/jeremymv2/env_pinning.

## What the heck is the "Role Cookbook Model" anyways?

First off, let's talk about blast radius, that will help to put things into 
context. Let's start with a scenario as follows:

_Scenario_: You have a Chef Server that's setup generally as-follows:
  * It has a `DEV,QA and PROD` environment, in which all nodes are assigned to.
  * It has roles such as `web_frontend, web_backend` etc... that nodes are also
  assigned to.
  * Finally, there are a standard set of cookbooks that are assigned to your
  roles.

  In our scenario, we want to roll out changes to our `web_backend` nodes,
  starting in `DEV` and moving to `PROD`. In `DEV`, we have 5 nodes, in `QA`, we
  have 10, and in `PROD` we have 50. There's also a cookbook called 
  `web_backend` that's currently at version `1.1`. This cookbook uses `v7.1.0` 
  of the `openssl` community cookbook currently, but it needs to be updated to
  use the latest version to fix a security flaw. 

  In this scenario, the `web_backend` cookbook has been designed with version
  pinning in mind. It has version `v7.1.0` currently explicitly pinned in it's 
  `metadata.rb`, and it will _only_ use that version of the `openssl` cookbook
  because of it. The `web_frontend` cookbook however doesn't have the `openssl`
  cookbook version pinned. This means that the default behaviour for anything
  assigned the `web_frontend` cookbook is to pull down the latest version of
  any dependant cookbooks, including `openssl`.

  So what happens when we update the `web_backend` cookbook to use the new
  version of `openssl`? Once that cookbook is published, along with the newer
  version of `openssl`, **ANY** cookbook that has a dependency on the `openssl` 
  cookbook that isnt explicity pinned to a version will pull down the latest
  version of `openssl`. This means that any node that has the `web_frontend` 
  cookbook assigned will update with the latest version of `openssl`. The 
  problems with this are obvious, the _intent_ was to make a change to nodes
  that are assigned to the `web_backend` role, however _unintentionally_ changes
  have now been made to the `web_frontend` nodes, along with any other nodes
  that have a dependency on `openssl` that haven't set the version explicitly.

So what's the solution to this?

### ... the Role Cookbook Model

The image below shows a simplified view of the Role Cookbook ecosystem, along
with the steps needed to update a Role Cookbook and publish it to your
environment. You'll notice the roles end in `_g`, that means "Green", we'll dive into that later as we discuss Blue/Green deployment scenarios.

![Role Cookbook Update Process](/images/role_cb_update_process.png)

1. A new version of the `openssl` cookbook has become available in
the community Chef Supermarket. The first step is to update the `metadata.rb` of
the Application Cookbook, in this case `web_backend_cb`. Also, as part of this 
step, you would bump the version of the `web_backend_cb` to a higher version.
2. The next step is to update your Role Cookbook with the new version of the
application cookbook you just bumped. Again, in doing so, you will in-turn bump
the version of the Role Cookbook.
3. The final step is to "flip the switch" on the Environment 'cookbook_versions'
pinnings. Once you do this, and push the Environment `.json` up to the Chef
Server, the changes will take affect.

Notice in the above model, the "blast radius" is limited to the `web_backend_g`
role _only_. This ensures that the changes we are making are only going to be
applied to the nodes that have been assigned that role.

### ... Now, one might say: "but, this doesn't stop unintentional changes to other nodes!"

And that is a very true statement! It doesn't, unless this process is applied to
every cookbook you and your team develop. By setting the environment pin to only
care about the Role Cookbook version, you're trusting that pinning will be 
handled in subsequent cookbooks. There are some simple methods that can be done
to automate this process, and even reject uploading a cookbook to the Chef 
Server unless it has dependency versions defined.

... more to come
