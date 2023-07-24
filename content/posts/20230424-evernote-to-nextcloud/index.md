super project, docker, well documented, out od the box. lists similar project on github, vorbild für open source projekt
oauth access to evernote
loads everything in a sqlite db but can also exported to enex format (xml based format by evernote)



╰─ docker run --rm -t -v "$PWD":/tmp -p 10500:10500 vzhd1701/evernote-backup:latest init-db --oauth
Unable to find image 'vzhd1701/evernote-backup:latest' locally
latest: Pulling from vzhd1701/evernote-backup
ab2f6dae3b54: Pull complete 
9411f38bb959: Pull complete 
39621572cdf6: Pull complete 
b0fe9d58182b: Pull complete 
d645f48b30f2: Pull complete 
Digest: sha256:413eee027a07e27e9ae329d7a3fd0762c00219da7bdfd25c45c0017b61469ca2
Status: Downloaded newer image for vzhd1701/evernote-backup:latest
Logging in to Evernote...
Opening authorization page...
If it didn't open automatically, please copy this URL into your browser:
https://www.evernote.com/OAuth.action?oauth_token=bulkbackup.187BA285559.687474703A2F2F6C6F63616C686F73743A31303530302F6F617574685F63616C6C6261636B.0A950F4A9D9112611742B1812EBDF8C1
Authorizing auth token, evernote backend...
Successfully authenticated as woolf44!
Current login will expire at 2023-04-26 20:43:59.
Initializing database en_backup.db...
Reading database en_backup.db...
Successfully initialized database for woolf44!

╭─ /tmp                                                                                         ✔  29s  10:44:01  
╰─ docker run --rm -t -v "$PWD":/tmp -p 10500:10500 vzhd1701/evernote-backup:latest sync           
Reading database en_backup.db...
Authorizing auth token, evernote backend...
Successfully authenticated as woolf44!
Current login will expire at 2023-04-26 20:43:59.
Syncing user notebooks...
  [####################################]  19248/19248          
651 note(s) to download...
Downloading 651 note(s)...
  [####################################]  651/651          
Updated or added notebooks: 7
Updated or added notes: 651
Expunged notebooks: 0
Expunged linked notebooks: 0
Expunged notes: 1
Synchronization completed!



docker run -t --rm -v "$PWD":/tmp -w /tmp wormi4ok/evernote2md:latest  inlupus.enex notes 