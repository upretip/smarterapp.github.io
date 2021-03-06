## Create AWS MongoDB Instance
* Create server instance to host the MongoDB instance that will support the component being deployed
  * Select an image with the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system
* Create or choose an AWS security group with the following ports for inbount TCP traffic (can be done during instance creation):
  * 22
  * 27017 - 27019
  * 28017 - 28018
* Remove `apparmor`:
  * `sudo /etc/init.d/apparmor stop`
  * `sudo update-rc.d -f apparmor remove`
  * `sudo apt-get --purge remove -y apparmor apparmor-utils libapparmor-perl libapparmor1`
* Update package manager:
  * `sudo apt-get update`
  * `sudo apt-get upgrade -y`
* Install packages to satisfy dependencies:
  * `sudo apt-get install -y ntp`
* Install MongoDB 2.4.9:
  * `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10`
  * `echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list`
  * `sudo apt-get update`
  * `sudo apt-get install mongodb-10gen=2.4.9`
* Pin the version of MongoDB so `apt-get` will not upgrade it:
  * `echo "mongodb-10gen hold" | sudo dpkg --set-selections`
* Configure MongoDB by copying the following into `/etc/mongodb.conf`:
* ***IMPORTANT:***{: style="color: #f00;"} *The config file below has `noauth=true` set.  This is a temporary configuration to allow for adding MongoDB user accounts.  This setting will be changed later in the checklist.*{: style="background-color: #ff0;"}

~~~~
# mongodb.conf

# Where to store the data.
dbpath=/var/lib/mongodb

#where to log
logpath=/var/log/mongodb/mongodb.log

logappend=true

#bind_ip = 127.0.0.1
bind_ip = 0.0.0.0
port = 27017

# Enable journaling, http://www.mongodb.org/display/DOCS/Journaling
journal=true

# Enables periodic logging of CPU utilization and I/O wait
#cpu = true

# Turn on/off security.  Off is currently the default
noauth = true
#auth = true

# Verbose logging output.
#verbose = true

# Inspect all client data for validity on receipt (useful for
# developing drivers)
#objcheck = true

# Enable db quota management
#quota = true

# Set oplogging level where n is
#   0=off (default)
#   1=W
#   2=R
#   3=both
#   7=W+some reads
#oplog = 0

# Diagnostic/debugging option
#nocursors = true

# Ignore query hints
#nohints = true

# Disable the HTTP interface (Defaults to localhost:27018).
#nohttpinterface = true

# Turns off server-side scripting.  This will result in greatly limited
# functionality
#noscripting = true

# Turns off table scans.  Any query that would do a table scan fails.
#notablescan = true

# Disable data file preallocation.
#noprealloc = true

# Specify .ns file size for new databases.
# nssize = <size>

# Accout token for Mongo monitoring server.
#mms-token = <token>

# Server name for Mongo monitoring server.
#mms-name = <server-name>

# Ping interval for Mongo monitoring server.
#mms-interval = <seconds>

# Replication Options

# in replicated mongo databases, specify here whether this is a slave or master
#slave = true
#source = master.example.com
# Slave only: specify a single database to replicate
#only = master.example.com
# or
#master = true
#source = slave.example.com

# Address of a server to pair with.
#pairwith = <server:port>
# Address of arbiter server.
#arbiter = <server:port>
# Automatically resync if slave data is stale
#autoresync
# Custom size for replication operation log.
#oplogSize = <MB>
# Size limit for in-memory storage of op ids.
#opIdMem = <bytes>
~~~~

* Restart MongoDB:
  * `sudo service mongodb restart`
* Add an administrative-level user to MongoDB:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>$ mongo admin
db.addUser({
    user:"mongo_admin",
    pwd:"[<span class="placeholder">choose a suitable password</span>]",
    roles:["dbAdminAnyDatabase","userAdminAnyDatabase","clusterAdmin","readWrite"]
});</code>
</pre>
</div>

* Update `/etc/mongodb.conf` to enable authentication:
  * Comment out the `noauth = true` line
  * Uncomment the `auth = true` line
* Example:

~~~~
# Turn on/off security.  Off is currently the default
#noauth = true
auth = true
~~~~

* Restart MongoDB:
  * `sudo service mongodb restart`
* Connect to MongoDB in the admin database:
  * `mongo admin -u mongo_admin -p `[*password for the mongo_admin user*{: style="color: #f00;"}]` --authenticationDatabase admin`
* Add a user for the component:

<div class="highlighter-rouge">
<pre class="highlight">
<code>use [<span class="placeholder">name of database</span>];
db.addUser({
    user:"[<span class="placeholder">name of user</span>]",
    pwd:"[<span class="placeholder">password for user</span>]",
    roles:["readWrite"]
});</code>
</pre>
</div>

* Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>use <span class="placeholder-example">progman</span>;
db.addUser({
    user:"<span class="placeholder-example">progman</span>",
    pwd:"<span class="placeholder-example">[redacted]</span>",
    roles:["readWrite"]
});</code>
</pre>
</div>

### Verify User Can Authenticate to MongoDB
* On the AWS instance hosting MongoDB, run the following commands:
  * `mongo admin -u mongo_admin -p '`[*The password for the mongo_admin user*{: style="color: #f00;"}]`' --authenticationDatabase admin`
  * `mongo [`*component database name*{: style="color: #f00;"}`] -u `[*Component user*{: style="color: #f00;"}] `-p '`[*The password for the component user*{: style="color: #f00;"}]`'`
* If successful, the prompt should appear as follows:

~~~~
MongoDB shell version: 2.4.9
connecting to: admin
>
~~~~