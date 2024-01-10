# DevOps-tooling-website-solution

sudo mount -t nfs -o rw,nosuid 172.31.34.162:/mnt/apps /var/www

lv--opt: UUID=96b2dac4-8bae-4933-b7f2-635fcafaf837
lv--apps: UUID=75e839cf-8491-469a-83d8-ab9af392b85b
lv--logs: UUID=cfbbe04c-a79a-4aa1-b1c9-7e3372ef0827

UUID=96b2dac4-8bae-4933-b7f2-635fcafaf837 /mnt/opt xfs defaults 0 0
UUID=75e839cf-8491-469a-83d8-ab9af392b85b /mnt/apps xfs defaults 0 0
UUID=cfbbe04c-a79a-4aa1-b1c9-7e3372ef0827 /mnt/logs xfs defaults 0 0

sudo setsebool -P httpd_execmem 1

sudo mount -t nfs -o rw,nosuid 172.31.34.162:/mnt/logs /var/log/httpd

mysql -h 172.31.36.84 -u webaccess -p tooling < tooling-db.sql



