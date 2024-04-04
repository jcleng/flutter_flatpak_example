- [Flutter Flatpak Example](#flutter-flatpak-example)
  - [Requirements](#requirements)
    - [VSCode dev container](#vscode-dev-container)
    - [GitHub actions](#github-actions)
    - [Manual Requirements](#manual-requirements)
  - [Instructions](#instructions)


# Flutter Flatpak Example


An example of how to package a Flutter application as a Flatpak for distribution
on Linux, using the default counter example app.

[Flatpak documentation](https://docs.flatpak.org/en/latest/index.html)



## Requirements

> Note: Building a flatpak should be done in a predictable environment or it may
> fail.

[Set up Flathub](https://flatpak.org/setup/), and choose one:

### VSCode dev container

Use the VSCode dev container provided in this repo that will run everything
through Docker

1. Install the [Dev
   Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
   extension

2. Open this directory in VSCode.

3. Accept the prompt to re-open in the dev container, or from the Command
   Palette search for `Reopen in Container`


### GitHub actions

> There is a [GitHub
> action](https://github.com/bilelmoussaoui/flatpak-github-actions) for this
> purpose, which is demonstrated in this repo. This action's page also lists the
> docker containers it uses.
>
> If you fork this example repo you can run the [example workflow](https://github.com/Merrit/flutter_flatpak_example/blob/main/.github/workflows/flatpak.yml), and
> install the `.flatpak` file it generates with `flatpak install <path-to-.flatpak>`.


### Manual Requirements

Install flatpak and flatpak-builder

On ubuntu:

```bash
sudo apt install flatpak flatpak-builder
```

Add the FlatHub repo:

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Install flatpak build dependencies:

```bash
flatpak install -y org.freedesktop.Sdk/x86_64/22.08
flatpak install -y org.freedesktop.Platform/x86_64/22.08
flatpak install -y flathub org.freedesktop.appstream-glib
```


## Instructions

We have two directories that each represent what would be separate git
repositories for a real project:

[counter_app](counter_app/) is the Flutter app, view the README there for info on
configuration and building.

[flathub_repo](flathub_repo/) is separate from the Flutter app and is where the Flatpak is
assembled, view the README there for info on configuration, building, and
publishing of the flatpak after building the `counter_app`.

- local debug

```shell
# ubuntu
docker run -itd -v /home/jcleng/work/:/home/jcleng/work/ \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY=$DISPLAY \
--ipc=host \
--privileged \
--name=flutter flutter:latest

cd /home/jcleng/work/flutter_flatpak_example
# build
cd counter_app/
/home/jcleng/work/flutter/flutter/bin/flutter pub get
ninja-build cmake clang pkg-config libgtk-3-dev
/home/jcleng/work/flutter/flutter/bin/flutter build linux --release
/home/jcleng/work/flutter/flutter/bin/flutter run -d linux --debug
# bin dir
build/linux/x64/release/bundle/flutter_flatpak_example

cd build/linux/x64/release/bundle
archiveName=FlutterApp-Linux-Portable.tar.gz
tar -czaf $archiveName ./*

# flatpak install
cd flathub_repo/
flatpak-builder --repo=repo --user --install --force-clean build-dir com.example.FlutterApp.yml

```
