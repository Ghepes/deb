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
