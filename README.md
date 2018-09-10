# Read and write Dash spi flash through wifi

### In setup mode Dash button has an open wifi named "Amazon ConfigureMe"

#### When the user configures it their wifi credentials are leaked on open wifi through http post

```
POST / HTTP/1.1
Host: 192.168.0.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:40.0) Gecko/20100101 Firefox/40.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.0.1/
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 306

amzn_devid=G030G00553360818&
amzn_macid=t%25C2F%25A8n%252F&
amzn_nonce=%2591%25F2%253E%25AC%25BE%25F7%2523Z4%25D8%25A0%25ED%2503%2582%25D9%25D7%25DA%2593%2501ir%2580X%25A9mY%25234f%252B%2502%2597&
amzn_fwver=v0.9.119&
amzn_ssid=Kobayashi+Heavy+Industries&
idValue=Kobayashi+Heavy+Industries&
amzn_pw=badpassword8
```

### Flash endpoint on web service can be seen in string dump of firmware
```
text/html
/flash
/token
/fresh.png
image/png
:%2x
hex:
```

#### And some commands
```
addr
sha1
cmd = %s; addr = %x; len = %x
read
write
erase
iread
update
update
%s line=%d msize=%dK
 blocks=%d
```

#### Running bad commands through the endpoint gives some feedback on the serial output
```
DMA overrun
cmd = *; addr = 0; len = 0
Found chirp at index 6400101
DMA overrun
Equalize: 24 16 26 22 
DMA overrun
Fail Fast Failure at index 1
```

#### Commands look like 
```
?cmd=Command&len=LengthInHex&addr=StartAddressInHex
```
read 1mb flash
http://192.168.0.1/flash?cmd=read&len=100000&addr=0

read 2mb spi flash rom
http://192.168.0.1/flash?cmd=read&len=200000&addr=0

#### write command works on spi rom but need to run erase first and wants to write in 256 byte chunks
```
http://192.168.0.1/flash?cmd=erase&len=100000&addr=0
```
### bash script to write binary to spi in chunks
```bash
mkdir -p chunks
cd chunks
split --bytes=256 myfirmware.bin chunks

add=0
while [ $add -lt 1048576 ]
do
    nextchunk=`ls -1|head -1`
    addr=`printf "%x" $add`
    echo "$addr $nextchunk"
    curl --data-binary @$nextchunk "http://192.168.0.1/flash?cmd=write&len=100&addr=$addr"
    rm -f $nextchunk
    add=$(( 0x${addr} + 0x100 ))
done
```

