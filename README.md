
# Torrent To Google Drive Downloader

This project allows you to download torrents directly to your Google Drive using Google Colab. It leverages the power of libtorrent and Google Drive integration to provide a seamless downloading experience.

## Features

- **Direct Download to Google Drive**: Save torrents directly to your Google Drive.
- **Support for Torrent Files and Magnet Links**: Add torrents using either torrent files or magnet links.
- **Interactive Progress Tracking**: Monitor the download progress with interactive widgets.

## Getting Started

### Prerequisites

- A Google account to access Google Drive.
- Basic knowledge of Python and Google Colab.

### Installation

1. **Clone the Repository**: Clone this repository to your local machine or open it directly in Google Colab.

2. **Install Dependencies**: Run the following command to install libtorrent:
    ```python
    !apt install python3-libtorrent
    ```

3. **Mount Google Drive**: Mount your Google Drive to save the downloaded files:
    ```python
    from google.colab import drive
    drive.mount("/content/drive")
    ```

### Usage

1. **Initialize Session**: Initialize the libtorrent session:
    ```python
    import libtorrent as lt
    ses = lt.session()
    ses.listen_on(6881, 6891)
    downloads = []
    ```

2. **Add Torrent File**: Upload and add a torrent file:
    ```python
    from google.colab import files
    source = files.upload()
    params = {
        "save_path": "/content/drive/My Drive/Torrent",
        "ti": lt.torrent_info(list(source.keys())[0]),
    }
    downloads.append(ses.add_torrent(params))
    ```

3. **Add Magnet Link**: Add a torrent using a magnet link:
    ```python
    params = {"save_path": "/content/drive/My Drive/Torrent"}
    while True:
        magnet_link = input("Enter Magnet Link Or Type Exit: ")
        if magnet_link.lower() == "exit":
            break
        downloads.append(lt.add_magnet_uri(ses, magnet_link, params))
    ```

4. **Start Download**: Start the download and monitor progress:
    ```python
    import time
    from IPython.display import display
    import ipywidgets as widgets

    state_str = ["queued", "checking", "downloading metadata", "downloading", "finished", "seeding", "allocating", "checking fastresume"]
    layout = widgets.Layout(width="auto")
    style = {"description_width": "initial"}
    download_bars = [widgets.FloatSlider(step=0.01, disabled=True, layout=layout, style=style) for _ in downloads]
    display(*download_bars)

    while downloads:
        next_shift = 0
        for index, download in enumerate(downloads[:]):
            bar = download_bars[index + next_shift]
            if not download.is_seed():
                s = download.status()
                bar.description = " ".join([download.name(), str(s.download_rate / 1000), "kB/s", state_str[s.state]])
                bar.value = s.progress * 100
            else:
                next_shift -= 1
                ses.remove_torrent(download)
                downloads.remove(download)
                bar.close()
                download_bars.remove(bar)
                print(download.name(), "complete")
        time.sleep(1)
    ```

## Important Note

To get more disk space, go to Runtime -> Change Runtime and select GPU as the Hardware Accelerator. This will give you around 384GB to download any torrent you want.

## License

This project is licensed under the MIT License.

---
