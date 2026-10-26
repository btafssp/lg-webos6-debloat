# lg-webos6-debloat

A boot script for a **rooted** LG webOS 6 TV that stops LG's ad and content-recognition (ACR) services, a few unused telemetry daemons, and the TV's built-in far-field microphone, while leaving voice search, ThinQ, casting and AirPlay working.

Tested on one LG OLED C1 (OLED C1PUB), webOS 6.5.3 build 47. It may not suit other models or firmware. Use it at your own risk.

## What it does

`60-debloat` is a [Homebrew Channel](https://github.com/webosbrew/webos-homebrew-channel) init.d hook. About a minute after each boot (and once more two minutes later, to catch services that start late), it:

- stops these systemd units and kills their processes: `contentminer`, `remotediag`, `dmost`, `rdxd`, `uploadd`;
- bind-mounts `/dev/null` over these binaries, so the luna bus can't restart them: `acr2` (automatic content recognition, `com.webos.service.acr`), `adoverlay-service` (ad overlays), `uploadd`, `sportsalarm`, `amazon-alexa-adapter`;
- locks the built-in far-field microphone (ALSA card 1, `WoV PDM Mic`, `/dev/snd/pcmC1D0c`).

`/etc` and `/usr` are read-only on webOS, so nothing on disk is changed. Everything is undone by removing the script and rebooting.

## Findings on webOS 6.5.3

- On webOS 6 the ACR service is `acr2` (`com.webos.service.acr`) and the ad overlay is `adoverlay-service`. Both are LS2 dynamic services, which the luna hub respawns on demand, so stopping them isn't enough; replacing the binary with `/dev/null` is.
- The far-field microphone was not held open by anything, and the Magic Remote's microphone does not use it: voice search still works with it locked.
- `contentminer`, `remotediag`, `dmost`, `rdxd` and `uploadd` exist as systemd units. `sportsalarm` and `amazon-alexa-adapter` also exist.
- Not present on this firmware: `objectdetection`, `mycar`, `familycare`, `wowplay`, `cups-lpd`, `service-logger`, `ninfod`.
- Deliberately left alone, and all still working afterwards: voice (`voiceconductor`, `voiceinput`, `com.webos.app.voice`, `nlpmanager`, `performer`), ThinQ and the LG account (`lg.thinqai.adapter`, `iot-client`, `iconnectivity`), `avahi`, AirPlay, app casting, Miracast, SSAP on ports 3000/3001, `sdx`, `xinetd`.
- Already failing before any change: `accountmanager`, `bugreportd`, `crashreportd`, `dropbear`.

## Install

On a rooted TV with Homebrew Channel, over SSH:

```sh
# copy 60-debloat to the TV first, then:
cp 60-debloat /var/lib/webosbrew/init.d/60-debloat
chmod +x /var/lib/webosbrew/init.d/60-debloat
/var/lib/webosbrew/init.d/60-debloat now   # apply immediately
cat /tmp/debloat.log
```

## Undo

```sh
rm /var/lib/webosbrew/init.d/60-debloat && reboot
```

To keep the rest but give the microphone back, delete the microphone lines at the end of `apply()` and reboot.

## Credit

The approach follows [own-your-glass](https://github.com/JulioFerrero/own-your-glass) by JulioFerrero (MIT), which targets webOS 10. That tool was not run here, and no code was copied from it; a few of its modules were reproduced by hand for webOS 6.

## License

MIT. © 2026 Brian T. Austin. See [LICENSE](LICENSE).
