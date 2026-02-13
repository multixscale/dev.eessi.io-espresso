# dev.eessi.io-espresso

Repository for pre-release builds of [ESPResSo](https://github.com/espressomd/espresso).
Builds are deployed to the [`dev.eessi.io`](https://www.eessi.io/docs/repositories/dev.eessi.io/) repository.

## Run workflow with the bot

See [our wiki](https://github.com/multixscale/dev.eessi.io-espresso/wiki).

## Run workflow locally

```sh
source /cvmfs/software.eessi.io/versions/2025.06/init/bash
module load EasyBuild/5.2.0
eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb \
   --include-easyblocks=easyconfigs/e/ESPResSo/espresso.py \
   --software-commit=dfda2d8 --max-parallel $(nproc)
module use ~/.local/easybuild/modules/all
module spider ESPResSo
module load ESPResSo/dfda2d8-foss-2025a
```

### Build missing dependencies locally

```sh
source /cvmfs/software.eessi.io/versions/2025.06/init/bash
module load EasyBuild/5.2.0
eb easybuild/easyconfigs/b/Boost.MPI/Boost.MPI-1.88.0-gompi-2025a.eb \
   --dump-test-report=Boost.MPI-1.88.0-gompi-2025a_$(date "+%Y%m%d").md \
   --robot --max-parallel $(nproc)
module use ~/.local/easybuild/modules/all
module spider Boost.MPI
eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb \
   --include-easyblocks=easyconfigs/e/ESPResSo/espresso.py \
   --software-commit=dfda2d8 --max-parallel $(nproc)
module load ESPResSo/dfda2d8-foss-2025a
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

## Migrate to a newer toolchain

- go to the top-level directory of your local `dev.eessi.io-espresso` fork
- select new `foss` toolchain version, e.g. 2025a, and create the corresponding
  easyconfig file using any available older easyconfig file from that toolchain,
  e.g. `cp easyconfigs/ESPResSo-foss-2023b-software-commit.eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb`
- select the software layer version that ships the selected toolchain version
  (see [EESSI versions](https://www.eessi.io/docs/repositories/versions/)),
  and create the corresponding easystack file with the updated EasyBuild version, e.g.
  `cp easystacks/software.eessi.io/2023.06/espresso-eb-5.1.0-dev.yml easystacks/software.eessi.io/2025.06/espresso-eb-5.1.2-dev.yml`
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
- source the chosen software layer version,
  e.g. `source /cvmfs/software.eessi.io/versions/2025.06/init/bash`
- load the chosen EasyBuild version,
  e.g. `module load EasyBuild/5.1.2`
- make sure that GCC and other modules are *not* loaded!
- make sure you are *not* in a Python environment!
- run `eb easyconfigs/ESPResSo-foss-2025a-software-commit.eb`

# Debugging

- add `configopts += ' -DMPIEXEC_PREFLAGS="--mca;vader;^btl,tcp,uct,ucx,smcuda,self;--mca;btl_base_verbose;40;--mca;mpi_cuda_support;0;--mca;mpi_abort_print_stack;enabled" -D CMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-O0 -g" '` to build in debug mode, activate stack traces on `MPI_Abort()`, disable MPI CUDA support, and choose a different transport mechanism
- add `runtest = '-V -L "unit_test"'` to display shell commands invoked by CTest
- module load GDB
- extract the unit test commands from the logfile and replace MPI option `-n 2` by `-n 2 xterm -fa 'Monospace' -fs 12 -e gdb --args` to run the test in an interactive GDB window (without `xterm`, the GDB might not start in interactive mode)
