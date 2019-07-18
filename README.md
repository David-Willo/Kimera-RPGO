RobustPGO
======================================

## Dependencies

*[GTSAM](https://bitbucket.org/gtborg/gtsam)*
(Note that a github version has also been released, but for the github version the flag `GTSAM_BUILD_WITH_MARCH_NATIVE` sometimes causes problems )

Clone GTSAM:   
```bash
cd
git clone git@bitbucket.org:gtborg/gtsam.git
```

Build
```bash
cd gtsam 
mkdir build
cd build
cmake .. -DGTSAM_POSE3_EXPMAP=ON -DGTSAM_ROT3_EXPMAP=ON
sudo make install
```
**Note:** 
The reason why we need EXPMAP is for the correct calculation of Jacobians. 
Enabling this and the `#define SLOW_BUT_CORRECT_BETWEENFACTOR` in LaserLoopCLosure.h are both important. Otherwise the default are some identity approximations for rotations, which works for some cases but fails for cases with manual loop closures, or artifacts. Note that `sudo make check` will partially fail because some unittests assume the EXPMAP flags to be off. 

## Build 
```bash
git clone git@github.com:MIT-SPARK/RobustPGO.git
cd RobustPGO
mkdir build
cd build
cmake ..
make 
```
If GTSAM Unittests give you trouble, you can do `cmake .. -DUNIT_TESTS=OFF`

## Usage 
This repository can be used as an optimization backend. A sample setup looks something like below. The default solver is `SOLVER=1` which uses LM for optimization. 
```cpp
// Set up 
std::shared_ptr<OutlierRemoval> pcm = std::make_shared<PCM<gtsam::Pose3>>(odom_threshold, pw_threshold, special_symbs);
pcm->setQuiet(); // optional: to turn off print messages for pcm

std::shared_ptr<RobustPGO> optimizer = std::make_shared<RobustPGO>(pcm, SOLVER, special_symbs);
pgo->setQuiet(); // optional: turn off print messages
//...
//...

// Run 
optimizer_->update(new_factor, new_values);
```
This can also be used as a standalone experimental tool. A read g2o function can be found in examples.
You can create a `log` folder inside the `build` folder.
The results (distance matrix and clique size data) from running the following script in the build folder will be saved to the `log` folder
```
# for 2D: 
./RpgoReadG2o 2d <g2o-file> <odom-check-threshold> <pcm-threshold> <optional:folder-to-save-g2o> <optional:v to toggle verbosity>

# for 3D 
./RpgoReadG2o 3d <g2o-file> <odom-check-threshold> <pcm-threshold> <optional:folder-to-save-g2o> <optional:v to toggle verbosity>
```

Example, do `./RpgoReadG2o 3d /home/user/Desktop/in.g2o 1.0 1.0 /home/user/Desktop/out.g2o v`
