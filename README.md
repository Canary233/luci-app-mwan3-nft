# mwan3 nftables feed

OpenWrt feed containing the nftables port of mwan3 and its LuCI app, with
local fixes for OpenWrt 25.12 / kernel 6.12 and coexistence with Tailscale.

Based on:

- <https://github.com/dl12345/mwan3> (mwan3 3.6.12)
- <https://github.com/dl12345/luci-app-mwan3> (LuCI frontend)

## Local changes

### `mwan3/Makefile`

- Fix the build dependency package name:
  `libnetfilter_conntrack` -> `libnetfilter-conntrack` (hyphenated package name).

### `mwan3/files/etc/init.d/mwan3`

- `stop_service()` now flushes **only the routing tables owned by mwan3's
  configured interfaces** (`mwan3_iface_tbl`) instead of every table with an
  ID `<= MWAN3_INTERFACE_MAX`.
- The old behaviour also flushed Tailscale's routing table **52** (which is
  below the default limit of 60), wiping all tailnet/subnet routes whenever
  mwan3 restarted and breaking subnet access from other Tailscale devices
  until `tailscaled` was restarted.

### `luci-app-mwan3/Makefile`

- Use `include $(TOPDIR)/feeds/luci/luci.mk` instead of `../../luci.mk` so the
  package builds correctly when loaded from a standalone feed.

## Usage

Add the feed to `feeds.conf`:

```
src-git mwan3nft https://github.com/Canary233/luci-app-mwan3-nft.git
```

Then replace the official iptables-based packages with this feed:

```sh
./scripts/feeds update mwan3nft
./scripts/feeds uninstall mwan3 luci-app-mwan3
./scripts/feeds install -p mwan3nft mwan3 luci-app-mwan3
make defconfig
```

Do not run `./scripts/feeds install -a` afterwards, otherwise the official
`mwan3` / `luci-app-mwan3` from the `packages` and `luci` feeds may be
installed again and conflict with this feed.
