# Multiple SSH Keys for Multiple Github Accounts
### How to configure multiple SSH Keys settings for different Github accounts and have your local development environment(s) set to work with them all.

It is a very common scenario: developers having to deal with two or more Github accounts, and/or several discentralized projects..

While Github helps keep each project organized and separated, most likely the developer will use the same machine to work with more than one Github account. As Github doesn't allow the same **SSH PUBLIC Key** to be added across accounts, the developer is forced to keep more than one set of **SSH Keys** (PRIVATE and PUBLIC Keys) in the same machine, then use the correct one to access each specific repository.

The usage of SSH is optional. For repositories accessible via **HTTPS** the **SSH Keys** are not required. As such, if you are not intending to use SSH, you don't need to read this tutorial.

I the same sense, this tutorial won't cover how to setup a **_single_ SSH Key**. It should be a basic task for any developer. However, if you don't know how to do it, I would strongly suggest you to read <a href="https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/" target="_blank">this page</a>. 

Finally, this tutorial only covers the process for Linux/Unix/Mac environments. For Windows<sup>tm</sup> environments I suggest you to <a href="http://www.bing.com/search?q=Setting+up+Git+and+GitHub+for+development+on+Windows&go=Submit&qs=n&form=QBLH&pq=setting+up+git+and+github+for+development+on+windows&sc=0-52&sp=-1&sk=&cvid=FC7B8D05F6D64B4D8B3B487FC862CD24" target="_blank">Bing</a> it.

So, let's get our handy dirty! The entire process can be done in three easy steps:


STEP 1 - Create your Public Keys
---------------------------------

Assuming you have <a href="https://git-scm.com/downloads" target="_blank">downloaded and installed the latest version of Git</a> and already <a href="https://git-scm.com/book/uz/v2/Customizing-Git-Git-Configuration#_git_config" target="_blank">configured it correctly</a>, now it is time to create your **SSH Keys**. Lets begin by creating your personal SSH Keys. Open the terminal and type the following: 

```shell
ssh-keygen -t rsa -C "your_email@youremail.com" <kbd>Enter</kbd>
```
Where it reads `your_email@youremail.com` you must enter the email associated with your Github personal account.

The SSH Key Generator should respond with a message indicating that you are about to create a SSH Key, then it will wait for you to enter the name of the file you want to use. Here is where the _tric_ begins. If you press <kbd>Enter</kbd> without entering any name the SSH Key Generator will create two files: `id_rsa` (the PRIVATE KEY) and `id_rsa.pub` (the PUBLIC KEY) in the indicated directory. You can leave it as it is for your primary SSH Key, of if you are to have only one SSH Key, but as we are targetting multiple SSH Keys, we must give it an unique name. I would suggest something like "`id_rsa_YourName`", so the keys will be created as `id_rsa_YourName` (the PRIVATE KEY) and `id_rsa_YourName.pub` (the PUBLIC KEY).

Once you type the desired name and press <kbd>Enter</kbd>, the SSH Key Generator will ask you for a passphrase. If you enter anything here you will be requested to typoe it each time you use the certificate (which won't be much different than type a password for a non SSH Key usage). So, keep it empty by pressing <kbd>Enter</kbd>. The SSH Key Generator will ask you to confirm the passphrase by re-entering it, so simply press <kbd>Enter</kbd> again and the process will be complete.

You should see a message saying `The key fingerprint is:` with a bunch of numbers and another message just after it saying `The key's randomart image is:` with an ASCII representation of an image. It means you key was created correctly.

Now, repeate the same process by typing `ssh-keygen -t rsa -C "email@business.com" <kbd>Enter</kbd>`, but now use the business emaill associated to the _other_ Github account. When asked by the name of the file you want to use you must enter a different name. I would suggest something like "`id_rsa_YourName_BusinessName`", so the keys will be created as `id_rsa_YourName_BusinessName` (the PRIVATE KEY) and `id_rsa_YourName_BusinessName.pub` (the PUBLIC KEY).

Once complete, you will have four files in your `~/.ssh/` directory:

```
id_rsa_YourName
id_rsa_YourName.pub
id_rsa_YourName_BusinessName
id_rsa_YourName_BusinessName.pub
```

**Important Remark About the `passphrases`:**  
If you decided to add a passphrase in your SSH Keys, but still wants to be able to use it without having to type it each time your SSH Key is invoked, you can add YOUR private key identities (from your ~/.ssh directory) to the authentication agent (ssh-agent) in your local machine. The authentication agent will remember the passphrases after you use it for the first time. This is a convenience option and work "per SSH Key", which means you will have to add one by one via command line using `ssh-add ~/.ssh/id_rsa_KEYNAME`. You can always list which of your SSH Keys are being "rememberd" by the ssh-agent by using the command `ssh-add -l`, and you can always erase all cached passphrases by using the command `ssh-add -D`.


STEP 2 - Create a SSH Config File
---------------------------------

The SSH configuration files allow you to create shortcuts for sshd server, including advanced ssh client options. You can configure your OpenSSH ssh client using various files as follows to save time and typing frequently used ssh client command line options such as port, user, hostname, identity-file and much more. Yoy can learn more about it <a href="http://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/" target="_blank">here</a>, <a href="https://sanctum.geek.nz/arabesque/uses-for-ssh-config/" target="_blank">here</a> and <a href="http://linux.die.net/man/5/ssh_config" target="_blank">here</a>.

In our case, as we are targeting Github, we can simply create a new file in the ~/.ssh/ directory named "config" (as indicated below):

```shell
cd ~/.ssh/
touch config
```

Then open it in a text editor (your choice: vi, vim, nano, gedit, kate, etc.) and paste in it the following:

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

Remember to fix the names to reflect the names you used when creating the SSH Keys.

### Important Remarks about the ` ~/.ssh/` directory and its files:

SSH requires specific permissions for its _files_ and _directories_, otherwise it won't work. Most of the time SSH errors or lack of functionality are caused by misset permissions. Here how to do it right:

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

To add your SSH Key simply select the button <kbd>New SSH key</kbd>, give it a name where it reads "Tile", then paste in "Key" the entire contents of the referenced PUBLIC KEY you want to have access to this account.

Simply repeat the same process for each Github account you want to have SSH Access, adding to the Github account ONLY the SSH Key that is pertaining for that account access.

You are done. Now lets use it! :)


Using your SSH Keys individually
================================

NO much to say here. Simply remember to refer to the proper host and the rest will simply work as desired. 

IN any standard configuration, Github is always accessed via `git@github.com`. If you look to the item `hoist` in the config file you just wrote, you will notice that for the `host = git@github.com` the SSH Key `~/.ssh/id_rsa_YourName` will be used. However, if you refer to `git@github.com-BusinessName`, the SSH Key that will be used will be `~/.ssh/id_rsa_YourName_BusinessName`.

That's it. Just reference the call to the correct `HostName` and let the SSH Config File do the rest.


Troubleshooting
---------------
Please refer to [Github SSH Issues](http://help.github.com/ssh-issues/) for common problems and solutions.

