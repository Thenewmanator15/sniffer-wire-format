# sniffer-wire-format

The framing two sniffer firmwares speak, as one compiled file rather than two
copies of it.

    frame.c  frame.h     10-byte header, CRC-16/CCITT-FALSE, frame types
    commands.h           commands from the host, statuses a board replies with

Consumed as a submodule at `shared/` by:

- [esp32c6-sniffer](https://github.com/Thenewmanator15/esp32c6-sniffer) —
  ESP32-C6 firmware, and the Wireshark plugin that speaks to both boards
- [nrf54l15-sniffer](https://github.com/Thenewmanator15/nrf54l15-sniffer) —
  nRF54L15 firmware

## Why this is a repository and not a copy

Two implementations of one wire format drift, and the drift is invisible: a
capture from one board decodes and a capture from the other does not, with
nothing to say which is wrong. Compiling one file into both firmwares makes
byte-identity a property of the build rather than something the tests have to
catch.

That only holds while both builds compile *this* file. A vendored copy would
put the guarantee back in CI, where it is a check that can be skipped rather
than a compile that cannot.

## The constraint

**This code may depend on nothing but `<stddef.h>`, `<stdint.h>` and
`<string.h>`.**

It is compiled by ESP-IDF and by Zephyr. An SDK header added here breaks the
other board's build — which is the mechanism working, not failing. If you find
yourself wanting `esp_err.h` or `zephyr/kernel.h` in this repository, the code
that needs it belongs in the firmware that has it.

## The authoritative definition is Python

The layouts here mirror `esp32c6_sniffer.framing` and `esp32c6_sniffer.batch`
in the host package, which are authoritative. This is the C side of that, held
to byte-identical output by the golden vectors in
[esp32c6-sniffer](https://github.com/Thenewmanator15/esp32c6-sniffer)'s
`host/tests/vectors/golden.json`.

Change the Python first, then this, then re-run those vectors.

## Updating a consumer

A submodule is pinned to a commit, so a firmware repository does not move
until it is told to:

```bash
cd shared
git pull origin main
cd ..
git add shared && git commit -m "shared: pull in <what changed>"
```

Both firmware repositories should be moved together when the wire format
changes, and their conformance builds re-run against the golden vectors.
Leaving one pinned behind is the drift this repository exists to prevent — it
is merely visible now, as a stale submodule, instead of silent.

## Licence

MIT. See [LICENSE](LICENSE).
