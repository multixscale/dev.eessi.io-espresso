# dev.eessi.io-espresso

Repository for pre-release builds of [ESPResSo](https://github.com/espressomd/espresso).
Builds are deployed to the [`dev.eessi.io`](https://www.eessi.io/docs/repositories/dev.eessi.io/) repository.

## Run workflow locally

```sh
source /cvmfs/software.eessi.io/versions/2023.06/init/bash
module load EasyBuild/5.1.1
eb easyconfigs/ESPResSo-foss-2023b-software-commit.eb \
   --include-easyblocks=easyconfigs/e/ESPResSo/espresso.py \
   --software-commit=11393df --max-parallel $(nproc)
module use ~/.local/easybuild/modules/all
module spider ESPResSo
module load ESPResSo/11393df-foss-2023b
```
