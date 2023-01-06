My new MacBook Air is proving to be all that I’d hoped, and it’s not just because of the fancy new M1 processors. Since I’m coming from a 2014 MacBook, I’m reaping the benefits of all the other advancements Apple has made to its laptop line in the intervening years, and prime among those is the incorporation of Touch ID: I’ve already enabled it for 1Password (what a lifesaver) and, thanks to a tip from [Twitter follower Josef](https://twitter.com/ifixcz/status/1329127972029263872?s=20), I can bring it to one of my other favorite places: the command line.

Josef pointed out that it’s relatively easy to add Touch ID support for `sudo`, the Terminal command that allows you to temporarily grant yourself the powers of the superuser, to do things that no mortal user can do! (Think of it as the command-line equivalent of typing your administrator password in that dialog box that pops up when you want to make a system-level change.)

The good news is that Apple has done most of the heavy lifting here by having built a pluggable authentication module (PAM) for Touch ID; all you need to do is essentially turn it on, which takes just a few simple steps.

First, open up Terminal. Navigate to the directory where the system stores the list of PAMs by typing `cd /etc/pam.d/` and open the `sudo` file there in your favorite command-line text editor.

[1](https://sixcolors.com/post/2020/11/quick-tip-enable-touch-id-for-sudo/#fn-18036-nano) (You can also always use a GUI editor like BBEdit too.) Note that if you open it via the command-line, you’ll need to use `sudo` itself to do so, since the file is (understandably) protected.

Once you’ve opened it, add the following below the first line (you’ll see the headers under which each of the entries goes):

> `auth sufficient pam_tid.so`

That line basically tells the `sudo` command that the Touch ID authentication module is sufficient to authorize the user, which is all you need to do.

![Sudo with Touch ID](https://i0.wp.com/sixcolors.com/wp-content/uploads/2020/11/sudo-touchid.png?ssl=1)

Save the file and you’re done! Now, the next time you use the `sudo` command, instead of being prompted for your password, you’ll get a dialog box asking you to authenticate with Touch ID, just as you would any other time you needed to authenticate. (And, as an extra bonus, if you choose to click the Enter Password, you’ll get prompted to use either the password _or_ your Apple Watch, if you have one.)

\[_**Dan Moren** is the East Coast Bureau Chief of Six Colors. You can find him on Twitter at @dmoren or reach him by email at dan@sixcolors.com. His latest novel, The Nova Incident, comes out in July and [is available to pre-order now](https://dmoren.com/the-nova-incident/), so do it!_\]

**If you appreciate articles like this one, support us by [becoming a Six Colors subscriber](https://sixcolors.com/subscribe/). Subscribers get access to an exclusive podcast, members-only stories, and a special community.**