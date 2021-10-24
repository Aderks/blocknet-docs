# Sia XCloud Services Guide (Video)


**Downloading and Configuring Blocknet wallet:**

* Download Blocknet client and fully sync

* Turn on XRouter `blocknetdx.conf` >> `xrouter=1`

* Modify `xrouter.conf` to lengthen timeout on SIA upload/download and explain `timeout=n-seconds` in which you will wait until you get a response from the host. This will make the Console/CLI unusable until the response is received or timeout has lapsed. Example `timeout=300` equal 5 minutes.

```
[xrs::SCrenterdownload]
timeout=300

[xrs::SCrenterupload]
timeout=300
```

* Users should only use smaller files (< 10mb) for now due to this service being so new.

**Using Sia Services:**

* Re-open wallet, sync fully. In console/CLI type `xrGetNetworkServices` until Sia (SC...) plugins are shown, this can take 5-10mins.

* Once Sia plugins are shown, type `xrService SCconsensus` and check if the host's Sia wallet is sync'd and matches (https://explore.sia.tech/), and that `"synced'= true`.

* Since the user is probably on https://explore.sia.tech/ grab a random Block TXID and use `SCconsensusblock`. Eg: `xrService SCconsensusblock 00ea31be2d5761140a7f446dcf198319ca697398d2b397747f2dfd307c65f171`

**Upload/Managing/Downloading Files:**

* Go to imgur/google images and find a random file. Make sure you grab the image address (Eg: https://i.imgur.com/4Kuye6W.jpg). Now `SCrenterupload` the file. `xrService SCrenterupload [file_url] [user_subfolder(s)] [file_name]`. Eg: `xrService SCrenterupload https://i.imgur.com/4Kuye6W.jpg userB/pictures landscape.jpg`

* Quick explanation of Siapath folder/file structure: `xcloud/your_username/random_folder/my.file`

* Run through SCrenter file management:

	* `xrService SCrenterfile userB/pictures landscape.jpg` (view single uploaded file info)
	* `xrService SCrenterfiles userB` (view all uploaded files by userB)
	  * Quick blurb about sharing your files. Share your `[user_subfolder]` name publicly with others if you wish to share your files. There is no encrpytion implemented (possible in the future)
	* `xrService SCrenterrename userB/pictures landscape.jpg outdoors.png` (rename any file)
	* `xrService SCrenterdelete userB/pictures landscape.jpg` (delete any file)

* Once file is uploaded to SIA (uploadprogress = 100 / redundancy=2.5+), then file is successfully stored on Sia's chain.

* Download your file back using SCrenterdownload (this call can take some time depending on file size). `xrService SCrenterdownload userB/pictures lanscape.jpg`. Response is a PixelDrain link, go to the link and file is available to view/download.