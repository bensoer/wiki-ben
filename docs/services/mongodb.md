## Install MongoDB

### Linux Mint
A walkthrough tutorial is available on the blog site: https://blog.bensoer.com/install-mongodb-on-linux-mint/

A brief breakdown of the steps will be listed here
<ol>
<li>Run the Following Command: <code>sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10</code></li>
<li>Edit the Follow Command: <br> 

```bash 
echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
```

</li>
<li>Replace <code>$(lsb_release -sc)</code> with the appropriate ubuntu version the Mint version you are installing MongoDB on is based off of. You can find a full list here : http://www.linuxmint.com/oldreleases.php</li>
<li> Update dependencies: <code>sudo apt-get update</code></li>
<li> Install Latest Version of MongoDB: <code>sudo apt-get install -y mongodb-org</code></li>
</ol>

### Ubuntu
To install on Ubuntu, the procedure is very simple. Executes the following commands in the listed order:

<ol>
<li><code>sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10</code></li>
<li>

```bash
echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
```

</li>
<li><code>sudo apt-get update</code></li>
<li><code>sudo apt-get install -y mongodb-org</code></li>
</ol>

**Note:** that this will install the latest version of mongo. To install a specific version see MongoDB Docs in the sources

## Resources
* https://blog.bensoer.com/install-mongodb-on-linux-mint/ <br>
* http://glenngeenen.be/untitled/ <br>
* https://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/ <br>
