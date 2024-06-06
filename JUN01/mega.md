
```bash
!wget https://github.com/t3rm1n4l/megacmd/releases/download/0.016/megacmd_0.016_linux_amd64.zip && unzip mega*

!echo '{"User": "chadsarago.n@gmail.com", "Password": "Karth123!", "DownloadWorkers": 4, "UploadWorkers": 4, "SkipSameSize": true, "Verbose": 1}' > ~/.megacmd.json



!wget https://github.com/t3rm1n4l/megacmd/releases/download/0.016/megacmd_0.016_linux_amd64.zip && unzip mega*

!chmod +x *

!echo '{"User": "gs.tatparth@gmail.com", "Password": "Karth12!", "DownloadWorkers": 4, "UploadWorkers": 4, "SkipSameSize": true, "Verbose": 1}' > ~/.megacmd.json

# https://github.com/t3rm1n4l/megacmd


```
### ~/.megacmd.json
```json
{"User": "gs.tatparth@gmail.com", "Password": "Karth12!", "DownloadWorkers": 4, "UploadWorkers": 4, "SkipSameSize": true, "Verbose": 1}
```

```bash
!./megacmd delete mega:/nse/x.txt
!./megacmd get mega:/nse/x.txt /content/x1.txt
!./megacmd list mega:/
!./megacmd put /content/x.txt mega:/nse/.txt
!./megacmd mkdir mega:/newdir
!./megacmd get mega:/mp.tar.bz2
!./megacmd list mega:/
!./megacmd sync /content mega:/testing
```