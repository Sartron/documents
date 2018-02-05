# FileProtect
FileProtect [is an EasyApache feature](https://documentation.cpanel.net/display/EA/Apache+Module%3A+FileProtect) created by cPanel in order to improve the security of a user's sites. This is done by modifying the permissions of a document root so that only Apache and the user may view its contents.  
FileProtect will automatically change the ownership on a document root to **user:nobody** as well as set permissions of **750**. An alternative permission that is sometimes set is **711**.

# Enabling FileProtect
The method used to enable FileProtect is different depending on whether or not the server is running EasyApache 4.

## EasyApache 3
Run `/usr/local/cpanel/scripts/easyapache` and recompile with the Apache module **Fileprotect** enabled.

## EasyApache 4
### GUI
Enable FileProtect within **WHM » Server Configuration » Tweak Settings » Security » Enable File Protect**.  

### CLI
Use the cPanel script `/usr/local/cpanel/scripts/enablefileprotect`. This **does not** toggle the option within WHM's _Tweak Settings_ interface.  
This is a simple one-liner to enable it if it isn't enabled already: `test -f /var/cpanel/fileprotect || /usr/local/cpanel/scripts/enablefileprotect`

# Using FileProtect
Run FileProtect for one user: `/usr/local/cpanel/3rdparty/bin/perl -mCpanel::FileProtect::Sync -e '() = Cpanel::FileProtect::Sync::sync_user_homedir('');'`  
Run FileProtect for all users: `/usr/local/cpanel/scripts/enablefileprotect`