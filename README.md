# dev.eessi.io-espresso

Repository for pre-release builds of [ESPResSo](https://github.com/espressomd/espresso).
Builds are deployed to the [`dev.eessi.io`](https://www.eessi.io/docs/repositories/dev.eessi.io/) repository.

## Run workflow locally

```sh
source /cvmfs/software.eessi.io/versions/2025.06/init/bash
module load EasyBuild/5.1.2
eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb \
   --include-easyblocks=easyconfigs/e/ESPResSo/espresso.py \
   --software-commit=dc87ede --max-parallel $(nproc)
module use ~/.local/easybuild/modules/all
module spider ESPResSo
module load ESPResSo/dc87ede-foss-2025a
```

### Build missing dependencies locally

```sh
source /cvmfs/software.eessi.io/versions/2025.06/init/bash
module load EasyBuild/5.1.2
eb easybuild/easyconfigs/b/Boost.MPI/Boost.MPI-1.88.0-gompi-2025a.eb \
   --robot --max-parallel $(nproc)
module use ~/.local/easybuild/modules/all
module spider Boost.MPI
eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb \
   --include-easyblocks=easyconfigs/e/ESPResSo/espresso.py \
   --software-commit=dc87ede --max-parallel $(nproc)
module load ESPResSo/dc87ede-foss-2025a
python -c 'import numpy, espressomd;print(f"numpy {numpy.__version__}, ESPResSo {espressomd.__version__}")'
```

### Contribute missing dependencies upstream

- contribute the easyconfig with a test report
  ([instructions](https://docs.easybuild.io/contributing/#contributing_easyconfigs),
  example: [easybuilders/easybuild-easyconfigs#24421](https://github.com/easybuilders/easybuild-easyconfigs/pull/24421))
- add easyconfig to the software layer
  ([instructions](https://www.eessi.io/docs/adding_software/opening_pr/),
  example: [EESSI/software-layer#1279](https://github.com/EESSI/software-layer/pull/1279))
- deploy new software ([instructions](https://www.eessi.io/docs/adding_software/adding_development_software/))

## Run workflow with the bot

See [our wiki](https://github.com/multixscale/dev.eessi.io-espresso/wiki).

## Migrate to a newer toolchain

- go to the top-level directory of your local easybuild-easyconfigs fork
- select new `foss` toolchain version, e.g. 2025a, and create the corresponding
  easyconfig file using any available older easyconfig file from that toolchain,
  e.g. `cp easyconfigs/ESPResSo-foss-2023b-software-commit.eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb`
- lookup GCC version for that toolchain version
  ([toolchain list](https://docs.easybuild.io/common-toolchains/#common_toolchains_overview))
- for each ESPResSo dependency, find the dependency version that matches
  the GCC version or toolchain version
   - use `module spider` followed by the dependency name,
     or the online database of supported software (example:
     [c/CMake](https://docs.easybuild.io/version-specific/supported-software/c/CMake/))
   - toolchains `GCCcore`, `GCC`, `gfbf`, and `gompi` are subsets of `foss`
     ([dependency tree](https://docs.easybuild.io/common-toolchains/#newest-generations-2022b-and-later)),
     therefore package variants matched by a `module spider` search from any
     of these base toolchains can be ingested by a `foss` easyconfig
- run `source /cvmfs/software.eessi.io/versions/2025.06/init/bash`
  (select the software layer version that contains the chosen toolchain version,
  see [EESSI versions](https://www.eessi.io/docs/repositories/versions/))
- run `module load EasyBuild/5.1.2`
  (select the EasyBuild version that contains the chosen toolchain version)
- make sure that GCC and other modules are *not* loaded!
- make sure you are *not* in a Python environment!
- run `eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb`
