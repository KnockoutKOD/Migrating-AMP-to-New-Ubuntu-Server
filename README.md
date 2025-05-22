# migrating-amp-ubuntu
Detailed instructions for migrating an entire AMP instance from one Ubuntu server to another.

# Required Prep Tasks
1. Update the AMP instance on the "old" server.
2. Assume the AMP user on the "old" server via SSH and using the command `sudo -i -u amp`.
3. Run `ampinstmgr stopall` and verify that all instances have stopped.
4. Return to your normal user by using `su username` and entering the password.
5. On the "new" server, SSH in and `sudo su`, then run `bash <(wget -qO- getamp.sh)` to freshly install AMP.
6. Run through the guided installation and configure as necessary. Try to keep it similar to what you had for the "old" server.
7. Once the installation is complete, assume the AMP user on the "new" server using the command `sudo -i -u amp`.
8. Run `ampinstmgr stopall`.
9. Switch back to your normal user with `su username` and enter the password, then run `service ampinstmgr stop` and authenticate to stop the service itself.
10. Use `sudo su` to access root, then `cd /home/amp` and then `mv .ampdata .ampdata-intial` to rename the freshly installed AMP's data folder to avoid conflicts during the upcoming data transfer.

# Data Transfer Tasks
12. Time to transfer the .ampdata folder from the "old" server to the new one. SSH into the "old" server and `sudo su` to have root access, then run the following command to perform a "secure copy": `scp -r /home/amp/.ampdata username@newserverIP:/home/amp/`

NOTE: You might need to change the ownership of the directory with `chown -hR username /home/amp` on both servers to get the transfer to work if you are getting a permissions denied error message. If you do need to chown, make sure to set it back to the amp user afterwards.

12. On the "new" server, while still running as root, run `ampinstmgr fixperms`, then `service ampinstmgr start`.
13. Assume the amp user via `sudo -i -u amp` and run `ampinstmgr reactivateall`. Enter the license key, wait for everything to reactivate.
14. Once you see "Done" in the SSH window for the "new" server, access the AMP web interface using ip-address:8080 (or whatever port you might've changed it to), log in, and verify that you can see your instances are there. I recommend rebooting the "new" server at this point uisng `sudo shutdown -r now`.

# Optional: Setting Up playit.gg Again
1. SSH into the "new" server and run this script by copying and pasting the entire thing into the SSH terminal window:
```
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | sudo tee /etc/apt/sources.list.d/playit-cloud.list
sudo apt update
sudo apt install playit
```
2. Run the following commands one at a time:
```
sudo systemctl enable playit
sudo systemctl start playit
playit setup
```
Copy the link provided from the `playit setup` command and set up the new agent. You probably already have existing tunnels, so when you are ready you can delete the old agent and migrate the tunnels to the new one.

Credit: ThePotatoKing via CubeCoders Discord (for the prep and data transfer foundational instructions), playit.gg documentation (for the playit.gg stuff)
