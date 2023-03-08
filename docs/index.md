# I2P Bote Protocol

* [clearnet](https://bote.readthedocs.io/en/latest/)
* [I2P](http://polistern.i2p/bote/)

**I2P-Bote** is a server-less encrypted [DHT Kademlia](https://en.wikipedia.org/wiki/Distributed_hash_table)-based email protocol.

## Versions

- [v5](v5/index.md): current version with fixed communication issues.
- [v6](v6/index.md): possible future version. Currently in draft state.

### Deprecated

- [v4](old/v4/introduction.md): Does not support new I2P addresses.

## Implementations

- [i2p.i2p-bote](https://github.com/i2p/i2p.i2p-bote)
    - Original implementation of the protocol as a plugin for Java I2P router
    - Protocol versions: v4
    - Status: looks abandoned
- [i2pboted](https://github.com/majestrate/i2pboted)
    - Standalone Go implementation
    - Protocol versions: v4
    - Status: looks abandoned
- [pboted](https://github.com/PurpleBote/pboted)
    - Standalone C++ implementation
    - Protocol versions: v4, v5
    - Status: active

## Resources

- [Documentation](https://bote.readthedocs.io/en/latest/) ([I2P](http://polistern.i2p/bote/))
- [Tickets/Issues](https://github.com/PurpleBote/bote/issues)

## Submitting changes

Please send a GitHub Pull Request to [I2P Bote documentation](https://github.com/PurpleBote/bote/pull/new/master) with a clear list of what you've done (read more about [pull requests](http://help.github.com/pull-requests/)).

Please make sure:

- All of your commits are atomic (one feature per commit).
- Log message for your commits is clear.  
  One-line messages are fine for small changes, but bigger changes should look like this:

```bash
$ git commit -m "A brief summary of the commit
>
> A paragraph describing what changed and its impact."
```

## Donations

The project is not intended to generate commercial benefits.

- **XMR**: `85P3aEXrYMn1YxnQaZSBWy6Ur6j9PVRxmCd3Ey1UanKAdKnhd2iYNdrEhNJ2JeUdcC8otSHogRTnydn4aMh8DwbSMs4N13Z`

## Also

If you have any questions about the code and style, you can find me in `[#dev]` channel on **ilita** IRC network in to I2P.

Sincerely yours,  
*polistern*
