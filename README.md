# Multiple SSH Keys for Multiple Github Accounts
### How to configure multiple SSH Keys settings for different Github accounts and have your local development environment(s) set to work with them all.

It is a very common scenario: developers having to deal with two or more Github accounts, and/or several discentralized projects..

While Github helps keep each project organized and separated, most likely the developer will use the same machine to work with more than one Github account. As Github doesn't allow the same **SSH PUBLIC Key** to be added across accounts, the developer is forced to keep more than one set of **SSH Keys** (PRIVATE and PUBLIC Keys) in the same machine, then use the correct one to access each specific repository.

The usage of SSH is optional. For repositories accessible via **HTTPS** the **SSH Keys** are not required. As such, if you are not intending to use SSH, you don't need to read this tutorial.

In the same sense, this tutorial won't cover how to setup a **_single_ SSH Key**. It should be a basic task for any developer. However, if you don't know how to do it, I strongly suggest you to read <a href="https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/" target="_blank">this page</a>. 

Finally, this tutorial only covers the process for **Linux**/**Unix**/**Mac** environments. For <b>Windows<sup>tm</sup></b> environments I suggest you to <a href="http://www.bing.com/search?q=Setting+up+Git+and+GitHub+for+development+on+Windows&go=Submit&qs=n&form=QBLH&pq=setting+up+git+and+github+for+development+on+windows&sc=0-52&sp=-1&sk=&cvid=FC7B8D05F6D64B4D8B3B487FC862CD24" target="_blank">Bing</a> it.

So, let's get our handy dirty! The entire process can be done in three easy steps:


STEP 1 - Create your Public Keys
---------------------------------

Assuming you have <a href="https://git-scm.com/downloads" target="_blank">downloaded and installed the latest version of Git</a> and already <a href="https://git-scm.com/book/uz/v2/Customizing-Git-Git-Configuration#_git_config" target="_blank">configured it correctly</a>, now it is time to create your **SSH Keys**. Lets begin by creating your personal **SSH Keys**. Open the terminal and enter the following: 

```shell
ssh-keygen -t rsa -C "your_email@youremail.com"
```
Where it reads **`your_email@youremail.com`** change it for the email associated with **your** Github personal account.

The **SSH Key Generator** should respond with a message indicating that you are about to create a set of **SSH Keys**, then it will wait for you to enter the name of the file you want to use. Here is where the _tric_ begins. If you press <kbd>Enter</kbd> without entering any name the **SSH Key Generator** will create two files: `id_rsa` (the PRIVATE KEY) and `id_rsa.pub` (the PUBLIC KEY) in the indicated directory. You can leave it as it is (blank) if it is for your primary or sinlge **SSH Key**. However, as we are targetting create multiple **SSH Keys** we should give it an unique name. To simplify management, I would suggest something like "`id_rsa_YourName`", so the **SSH Key** files will be created as: `id_rsa_YourName` (the PRIVATE KEY) and `id_rsa_YourName.pub` (the PUBLIC KEY, which will have a `.pub` extension).

Once you type the desired name and press <kbd>Enter</kbd>, the **SSH Key Generator** will ask you for a **_passphrase_**. If you enter anything here you will be asked to re-enter it each time you use the **SSH Key** (and we don't want that). So, keep it empty by pressing <kbd>Enter</kbd>. The **SSH Key Generator** will ask you to confirm the **_passphrase_**. Simply press <kbd>Enter</kbd> once again for the process to complete.

You should see a message saying `The key fingerprint is:` with a bunch of _numbers_, followed by another message saying `The key's randomart image is:`, with an ASCII representation of an image under it. It means you **SSH Keys** were created correctly.

Now, let's create the second set of **SSH Keys** we will use for the _other_ Github account. Simply repeat the same process by typing `ssh-keygen -t rsa -C "email@business.com"` followed by <kbd>Enter</kbd>, but this time use your _business email_ or whatever email that is associated to the _other_ Github account. Then, when it asks for the **SSH Key** _file name_, use a name different than the previous **SSH Key** created. I would suggest something like "`id_rsa_YourName_BusinessName`", which will created the **SSH Keys** as `id_rsa_YourName_BusinessName` (the PRIVATE KEY) and `id_rsa_YourName_BusinessName.pub` (the PUBLIC KEY).

Once complete, you will have four files in your `~/.ssh/` directory (two for each **SSH Key Set**):

```
id_rsa_YourName
id_rsa_YourName.pub
id_rsa_YourName_BusinessName
id_rsa_YourName_BusinessName.pub
```

**Important Remark About the `passphrases`:**  
If you decided to add a **_passphrase_** in your **SSH Keys**, but still want to be able to use it without having to enter it each time your **SSH Key** is invoked, you can add _YOUR_ private key identities (from your `~/.ssh` directory) to the _authentication agent_ (`ssh-agent`), for your local machine only. The _authentication agent_ will remember the **_passphrases_** after you use it once. This is a convenience option, and works "per **SSH Key**", which means you will have to add one by one via command line using `ssh-add ~/.ssh/id_rsa_KEYNAME`. You can always list which of your **SSH Keys** are being "remembered" by the `ssh-agent` using the command: `ssh-add -l` <kbd>Enter</kbd>. To erase the cached **_passphrases_** on your system use the command: `ssh-add -D` <kbd>Enter</kbd>.


STEP 2 - Create a SSH Config File
---------------------------------

The _SSH configuration files_ allow you to create shortcuts for `sshd server`, including advanced _SSH Client Options_ to it (if needed). You can have various configuration files for the same _SSH Client_, each one for a different purpose and/or with specific command line options, such as: port, user, hostname, identity-file and much more. You can learn more about it <a href="http://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/" target="_blank">here</a>, <a href="https://sanctum.geek.nz/arabesque/uses-for-ssh-config/" target="_blank">here</a> and <a href="http://linux.die.net/man/5/ssh_config" target="_blank">here</a>.

In our case, as we are targeting Github multiple accounts, we can simply create a new file in the `~/.ssh/` directory and name it "**config**" (or "**github-config**" if you prefer). Simply enter the following commands:

```shell
cd ~/.ssh/
touch config
```

It will create an empty file with proper permissions. Then, open the newly created file in a text editor of your choice (vi, vim, nano, gedit, kate, etc.) and paste the following _script_ in it:

```bash
# My Personal Github Account Access
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_YourName

# My Business Github Account Access
Host github.com-BusinessName
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_YourName_BusinessName
```

**NOTE:** Remember to fix the **SSH Key _File Names_** to reflect the ones you used when creating the **SSH Keys** before.

Save the file and you are done setting the computer side.

### Important Remarks about the `~/.ssh/` directory and the **SSH Key** files:

Most of the time people fail using **SSH Keys** or get access errors because of bad permissions. SSH requires specific permissions for its _files_ and _directories_, otherwise it won't work. In some systems it may be more or less strict, but we will based our teachings here what the **_SSH Standards_** define. Here the basic recipe of how it should be done to function properly:

- The `~/.ssh` directory permissions must be **`700 (drwx------)`**.  
  You can use the following command to achieve it:
  
  `chmod 700 ~/.ssh` <kbd>Enter</kbd>
  
- The public key file(s), the ones with `.pub` extesion, must be set with **`644 (-rw-r--r--)`** permissions.  
  You can use the following command to achieve it:
  
  `chmod 644 ~/.ssh/[Filename.pub]` <kbd>Enter</kbd>
  
- The private key, the `id_rsa` file, the ones with **no** extension, must be set with **`600 (-rw-------)`** permissions.  
  You can use the following command toachieve it:
  
  `chmod 600 ~/.ssh/[Filename]` <kbd>Enter</kbd>
  


STEP 3 - Saving the SSH Keys on Github
--------------------------------------

This is the simplest part. Just login on Github and point the browser to: `https://github.com/settings/keys`. If you prefer you can navigate to this page by clicking on your Image Menu (top-right of screen) and select the option `Your Profile`. Then click at the button where it says <kbd>Edit profie</kbd> and select the option `SSH and GPG keys`. 

To add your **SSH Key** simply select the button <kbd>New SSH key</kbd>, give it a name where it reads "Tile", then paste in "Key" the entire contents of the referenced PUBLIC KEY you want to have access to this account.

Simply repeat the same process for each Github account you want to have SSH Access, adding to the Github account ONLY the **SSH Key** that is pertaining for that account access.

You are done. Now lets use it! :)


Using your SSH Keys individually
================================

No much to say here. Simply remember to refer to the proper host and the rest will simply work as desired. 

In any standard configuration, Github is always accessed via `git@github.com`. If you look to the item `hoist` in the config file you just wrote, you will notice that for the `host = git@github.com` the **SSH Key** `~/.ssh/id_rsa_YourName` will be used. However, if you refer to `git@github.com-BusinessName`, the **SSH Key** that will be used will be `~/.ssh/id_rsa_YourName_BusinessName`.

That's it. Just refer the call to the **correct `HostName`** and let the _SSH Config File_ work out the rest.


Troubleshooting
---------------
Please refer to [Github SSH Issues](http://help.github.com/ssh-issues/) for common problems and solutions.

