# retoc

> [!IMPORTANT]
> This is a fork of [trumank/retoc](https://github.com/trumank/retoc) for building S.T.A.L.K.E.R. 2 mods in the
> layout GSC's Zone Kit produces, and is the `retoc` that [ZonePak](https://github.com/SSyl/STALKER2-ZonePak) expects. It
> changes `to-zen` as described under [Changes from upstream](#changes-from-upstream) and otherwise matches
> upstream commit `885a8da`. For anything else, use upstream retoc.

CLI tool for packing/unpacking Unreal Engine IoStore containers (.utoc/.ucas) as
well as converting between Zen assets and Legacy assets (found in .pak containers).

## cli
```console
Usage: retoc.exe --aes-key <AES_KEY> [OPTIONS] <COMMAND>

Commands:
  manifest              Extract manifest from .utoc
  info                  Show container info
  list                  List fils in .utoc (directory index)
  verify                Verify IO Store container
  unpack                Extracts chunks (files) from .utoc
  unpack-raw            Extracts raw chunks from container
  pack-raw              Packs directory of raw chunks into container
  to-legacy             Converts asests and shaders from Zen to Legacy
  to-zen                Converts assets and shaders from Legacy to Zen
  get                   Get chunk by index and write to stdout
  dump-test             Dump test
  gen-script-objects    Generate script objects global container from UE reflection data .jmap see https://github.com/trumank/jmap
  print-script-objects  Print script objects from container
  help                  Print this message or the help of the given subcommand(s)

Options:
  -a, --aes-key <AES_KEY>

      --override-container-header-version <OVERRIDE_CONTAINER_HEADER_VERSION>
          [possible values: PreInitial, Initial, LocalizedPackages, OptionalSegmentPackages, NoExportInfo, SoftPackageReferences, SoftPackageReferencesOffset]
      --override-toc-version <OVERRIDE_TOC_VERSION>
          [possible values: Invalid, Initial, DirectoryIndex, PartitionSize, PerfectHash, PerfectHashWithOverflow, OnDemandMetaData, RemovedOnDemandMetaData, ReplaceIoChunkHashWithIoHash]
  -h, --help
          Print help
  -V, --version
          Print version
```

### overrides
Overrides may be required to extract content from certain games (usually pre-5.0 where IOStore version was not well defined). When repacking content to generate mods, the same override must be applied.

### to-legacy
```console
$ ls AbioticFactor/Content/Paks
global.ucas
global.utoc
pakchunk0-Windows.pak
pakchunk0-Windows.ucas
pakchunk0-Windows.utoc

$ retoc to-legacy AbioticFactor/Content/Paks legacy_P.pak
Detected package version: FPackageFileVersion(UE4: 522, UE5: 1012), EZenPackageVersion: 3
[00:00:06] ########################################   22522/22522
Extracted 22522 (0 failed) legacy assets to "legacy_P.pak"
Shader Library ShaderArchive-Global-PCD3D_SM5-PCD3D_SM5 statistics: Shared Shaders: 10; Unique Shaders: 4324; Detached Shaders: 0; Shader Maps: 339 (referenced by 0 packages), Uncompressed Size: 68MB, Compressed Size: 15MB, Compression Ratio: 445%
Shader Library ShaderArchive-AbioticFactor_Chunk0-PCD3D_SM5-PCD3D_SM5 statistics: Shared Shaders: 1941; Unique Shaders: 12327; Detached Shaders: 0; Shader Maps: 819 (referenced by 1721 packages), Uncompressed Size: 138MB, Compressed Size: 43MB, Compression Ratio: 322%
Extracted 2 shader code libraries to "legacy_P.pak"
```

### to-zen
```console
$ retoc to-zen legacy_P.pak iostore.utoc --version UE5_4
converting shader library "AbioticFactor/Content/ShaderArchive-AbioticFactor_Chunk0-PCD3D_SM5-PCD3D_SM5.ushaderbytecode"
Shader Library ShaderArchive-AbioticFactor_Chunk0-PCD3D_SM5-PCD3D_SM5 statistics: Shader Groups: 1115, Shader Maps: 819, Uncompressed Size: 138MB, Original Compressed Size: 43MB, Total Group Compressed Size: 10MB, Recompression Ratio: 397%, Total Compression Ratio: 1277%
converting shader library "AbioticFactor/Content/ShaderArchive-Global-PCD3D_SM5-PCD3D_SM5.ushaderbytecode"
Shader Library ShaderArchive-Global-PCD3D_SM5-PCD3D_SM5 statistics: Shader Groups: 392, Shader Maps: 339, Uncompressed Size: 68MB, Original Compressed Size: 15MB, Total Group Compressed Size: 6MB, Recompression Ratio: 222%, Total Compression Ratio: 987%
[00:00:02] ########################################   22522/22522

$ ls
legacy_P.pak
iostore.ucas
iostore.utoc
iostore.pak
...
```

## Changes from upstream

Zone Kit mods put their text database in an `OverrideContent` container. Its packages sit under `/Game/` on disk
but are named `/<Mod>/...`, and the container mounts at `../../../Stalker2/Content/`. Stock `to-zen` can write
neither, so this fork reads two environment variables:

- `RETOC_TOC_MOUNT_POINT` sets the mount point `to-zen` writes into the `.utoc` directory index, for example
  `../../../Stalker2/Content/`. Asset paths inside the container still resolve from `../../../`.
- `RETOC_PACKAGE_REDIRECTS` adds package redirects to the container header, as `source=target` pairs separated by
  `;`, for example `/Game/Autogenerated_1793294562_LocalizationDatabase=/MyMod/Autogenerated_1793294562_LocalizationDatabase`.

Container headers also start with a redirect name map of the `Container` type instead of the default.

## compatibility
Unreal Engine versions 5.3+ are well supported and can convert entire games to
.pak and have them run without issue. Lack of dependency information in versions
prior to 5.3 may result in games failing to load a handful of assets, preventing
them from running from .pak, but should not be an issue for modding purposes.

## credits
- [Archengius](https://github.com/Archengius): writing all of the asset conversion code
- [LongerWarrior](https://github.com/LongerWarrior): debugging complex conversion issues and testing against many games
- [ThePsyjo](https://github.com/ThePsyjo): the Zone Kit patch, from [stalker2-worth-per-weight](https://github.com/ThePsyjo/stalker2-worth-per-weight)
