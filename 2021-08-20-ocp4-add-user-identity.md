# Linking an existing Identity to a User on OpenShift 4

## Configuring Azure AD for ARO

In my current project we have an ARO 4 cluster that currently has the project's Active Directory configured for login. (cf.)[https://examples.openshift.pub/cluster-configuration/authentication/activedirectory-ldap/]
Now I was intrigued whether our Azure AD (where only the Ops team has accounts) could be connected and found [Azure Active Directory Integration With OpenShift 4 ARO 4](https://cloud.redhat.com/blog/openshift-blog-aro-aad).

I basically skipped everything from the guid as cluster already exists.
I made the changes to the auto-generated ARO Service Principal via Portal (`aro-<generated id>` under <em>Azure Active Directory</em> &rarr; <em>App registrations</em>).

I added the redirect URI as described in the guide and then configured the <em>Optional Claims</em> (`upn`, `email`) via Portal (under <em>Token Configuration</em>).
Azure Portal asked whether it should grant the application the appropriate permissions for the claims, so I didn't need to change the permissions myself.
(i.e. <em>Add API Permission to the Service Principal</em> was not necessary.)

## Linking existing identies

After successfully logging in via AAD, my new user was there. But I had no permissions, obviously.

I already have a user that is in groups synced from project-AD, but with different name.

After checking whether maybe Azure AD roles could be used, [but that is not supported yet](https://access.redhat.com/solutions/5239211), I decided to go another route:
Assigning the identity from AAD to my (project-)AD User.

After playing a little bit around, locking my account out of the cluster, needing to revert to kubeadmin I found the (manual) steps necessary:

1. locate your AAD identity name: `oc get identity`
2. add the identity's name to your existing user: `oc edit user <your user>` (under identities)
3. make note of your user's `uid`.
4. change `user` in the AAD identity to match your user's `name` and `uid`: `oc edit identity <aad identity>`
5. delete the generated AAD user (not sure if really necessary though)
6. et voila, your AAD identity is now mapped to your other user.

