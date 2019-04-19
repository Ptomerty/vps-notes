# Install Kappbot

Download:
```bash
git clone https://github.com/Ptomerty/Kappbot.git
cd Kappbot
git checkout dev
npm install
npm install -g forever
sudo ln -s $(which forever) /usr/bin/forever
```

Drop in `~/start.sh` file:
```bash
#!/bin/bash

cd ~/Kappbot
/usr/bin/forever start -c /usr/bin/node Kappbot.js
```

Add to `crontab`:
```
@daily /usr/bin/forever restartall
@reboot /bin/sh /home/ptomerty/start.sh
```