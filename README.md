# deb
Debian files terminal commands


# install PACKAGES
````
sudo apt update
sudo apt install dpkg-dev apt-utils gnupg -y
sudo apt install dpkg-dev fakeroot -y
````

# opening the deb file

````
dpkg-deb -R xzy_wromo_1.2.3-1+clp-xzy_all.deb folder_new/
````

# Rebuild the .deb
````
dpkg-deb -b folder_new/ xzy_wromo_1.2.3-1+clp-xzy_all.deb
````
___

# Other comments
Start new secret key
````
gpg --gen-key
````
Export the generate key to key.gpg
````
gpg --armor --export xyz@example.com > key.gpg
````
write a key.gpg file to a protected folder `depends on the case`
````
gpg --armor --export xyz@example.com | sudo tee key.gpg > /dev/null
````

RE-editing your key:
````
gpg --edit-key xyz@example.com
````

A special gpg> prompt will appear. Type the command:
````
expire
````
The system will ask you how long you want it to be valid. You can enter, for example:

5y (for 5 years)

10y (for 10 years)

0 (to never expire)

Confirm by entering y

After saving the new settings, run the command from point 1 (gpg --armor ... | sudo tee ...) to export the new updated key over the key.gpg file.
___

# Generating echo key for xyz.sh file
````
sha256sum xyz.sh
````
___

# Generate NEW Hash in the project
# Global execution for generation and signing
# in more complex project situations the following code _ Terminal
````
# 1. UPDATE THE .gz ARCHIVES (This solves the "Hash Sum mismatch" error)
echo "Regenerating the Packages.gz files..."
find . -type f -name "Packages" -exec sh -c 'gzip -9 -c "$1" > "$1.gz"' _ {} \;
echo "The .gz files have been updated!"

# 2. Run the loop to recalculate the hashes and sign
for codename in */; do
  codename=${codename%/}
  
  echo "========================================"
  echo "Processing the system: $codename"
  echo "========================================"
  
  cd "$codename"

  # Create the configuration (PUT THE CORRECT DOMAIN HERE: v1, v2 or v3 depending on which folder you are in!)
  cat <<EOF > release.conf
APT::FTPArchive::Release::Origin "xyz.example.com";
APT::FTPArchive::Release::Label "NameXYZ";
APT::FTPArchive::Release::Suite "$codename";
APT::FTPArchive::Release::Codename "$codename";
APT::FTPArchive::Release::Architectures "amd64 arm64";
APT::FTPArchive::Release::Components "ADD_ALL_FOLDER_NAME main main-test main-dev nginx nginx-dev nginx-test php-7.1 php-7.2 php-7.3 php-7.4 php-8.0 php-8.1 php-8.2 php-8.3 php-8.4 php-8.5 proftpd percona-server-server-8.0 varnish-7";
APT::FTPArchive::Release::Description "NameXYZ Next Repository New";
EOF

  # We generate the new Release 
  apt-ftparchive -c release.conf release . > Release
  rm -f release.conf InRelease Release.gpg

  # Signing 
  gpg --default-key xyz@example.com --clearsign -o InRelease Release
  gpg --default-key xyz@example.com -abs -o Release.gpg Release

  echo "✔ $codename has been validated and signed!"
  cd ..
done

````

# or separately digital signature (Seal)
````
# We generate the InRelease file
gpg --default-key xyz@example.com --clearsign -o InRelease Release

# Generate the Release.gpg file
gpg --default-key xyz@example.com -abs -o Release.gpg Release
````


# Replace text "Packages" in all Packages files (Automatic)
````
find . -type f -name "Packages" -exec sed -i 's/PackName/NewPackName/g' {} +
````

___
