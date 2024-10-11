# UAX DLTS GNU/Linux - Live System

## How to Rebuild

Rebuilding the live ISO must be done using a `debian:12-slim` Docker container with privileges enabled.

1. Spawn the Debian container:

    ```
    docker run -ti --name iso-build --privileged debian:12-slim
    ```

2. Inside the Debian container, install the necessary packages:

    ```
    apt update
    apt install --no-install-recommends live-build live-config git cpio ca-certificates busybox
    ```

3. Create a workspace directory, e.g., `/opt/live`:

    ```
    mkdir -p /opt/live
    cd /opt/live
    ```

4. Clone this repository to the workspace

    ```
    git clone https://github.com/antrusd/uax-dlts.git
    cd uax-dlts
    ```

5. Make some minor adjustments since the `Live system` boot menu label is not configurable:

    ```
    sed -i 's|Live system |UAX DLTS v0.1 GNU/Linux |g' /usr/lib/live/build/binary_grub_cfg
    sed -i 's|Live system |UAX DLTS v0.1 GNU/Linux |g' /usr/share/live/build/bootloaders/syslinux_common/live.cfg.in
    ```

6. Initialize the live config. At this stage, you can customize anything inside the `config` directory based on Debian Live Manual [here](https://live-team.pages.debian.net/live-manual/html/live-manual/customization-overview.en.html). The following command needs to be repeated each time you want to run/re-run the build. You can set `--cache` to `true` to cache some files, but the caching process may take a long time, so your results may vary.

    ```
    lb clean && lb config \
                --mirror-bootstrap http://deb.debian.org/debian/ \
                --mirror-binary http://deb.debian.org/debian/ \
                --mirror-binary-security http://deb.debian.org/debian-security/ \
                --distribution bookworm \
                --binary-image iso-hybrid \
                --apt-source-archives false \
                --archive-areas 'main non-free-firmware' \
                --parent-archive-areas 'main non-free-firmware' \
                --security true \
                --system live \
                --zsync false \
                --architecture amd64 \
                --cache false \
                --bootloaders syslinux,grub-efi
    ```

7. Run the build

    ```
    lb build
    ```

8. The ISO can be found at `/opt/live/live-image-amd64.hybrid.iso`. To copy the ISO to your local, run the following command from another terminal:

    ```
    docker cp iso-build:/opt/live/live-image-amd64.hybrid.iso ./
    ```

9. Test the ISO using qemu

    ```
    qemu -cdrom live-image-amd64.hybrid.iso -m 1g
    ```
