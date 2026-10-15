# File Upload Vulnerabilities — Prevention & Remediation Guide

## 1. Gold Standard Architecture for Secure File Uploads

1. **Store Files Outside the Web Root:**
   Store uploaded files in a directory completely outside the server's public document root (e.g., `/var/storage/uploads/` instead of `/var/www/html/uploads/`), or use object storage (AWS S3, Google Cloud Storage, Azure Blob).
2. **Serve Files via an Application Handler:**
   Never allow direct URL access to uploaded files. Use a dedicated script (`/download?fileId=123`) that sets `Content-Type` strictly based on validated MIME types and adds `Content-Disposition: attachment`.
3. **Disable Script Execution in Upload Directories:**
   Configure web servers to never execute scripts in upload folders:
   ```apache
   # In Apache directory config
   <Directory "/var/www/uploads">
       php_admin_flag engine off
       Options -ExecCGI
       AddHandler cgi-script .php .pl .py .jsp .asp
       RemoveHandler .php .phtml
   </Directory>
   ```
   ```nginx
   # In Nginx server block
   location /uploads/ {
       location ~ \.(php|phtml|phar)$ {
           deny all;
       }
   }
   ```
4. **Randomize Filenames:**
   Generate a random UUIDv4 for every saved file and discard the user-supplied filename and extension entirely.
5. **Re-encode Images:**
   Pass uploaded images through an image processing library (like ImageMagick or PIL) to re-render the pixels, stripping polyglot PHP payloads and malicious EXIF comments.
