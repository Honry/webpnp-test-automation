This branch is specific for developers' development testing purpose, which does not connect report server but supports more workloads automation, bisect automation, running multiple chrome builds at one time, etc..

This is a test automation framework based on [Playwright](https://github.com/microsoft/playwright), supports key Web workloads automation testing on Chrome browser across multiple platforms, as well as automatically generating test result and sending test report via mail.

## Support

- Platforms: Windows, Linux
- Workloads: Speedometer2, WebXPRT3, Unity3D, JetStream2, WebXPRT2015, BaseMark, Aquarium, TensorFlow local benchmark(WASM/SIMD/WebGL backends)

## Dependencies

- [Node.js](https://nodejs.org/en/), recommend to use Node.js LTS, current this tool is based on Node.js (12.17.0 LTS).

## Workflow
The automation test mainly takes the following actions:
* Get the system information like CPU, Memory, OS.
* Launch the Chrome browser and read the GPU information and browser version.
* Launch the chrome and run each workload defined in config.json for several times. The test results of all arounds are recorded.
* Choose the middle value among the test arounds and store the device information and all the results of this workload test
  to `./results/{platform}/{workload}` directory(If it does not exists, create it). The files are named as `{data}_{CPU}_{Browser}.json`.
* Generate a single report with summary of test results and send email to configured mail list. If there're any errors occured, send the errors information.

## Usage
- Go to this folder
- ```javascript
  npm install
  ```
- Config test plan via config.json:
  1. Set test target in `workloads` fields, pls. don't edit the workload name while you can change the workload's url and running times via `url` and `run_times` fields respectively. You can also remove some of these workloads to in order to run single workload testing.
  1. To support both Windows and Linux platforms, `win_chrome_path` and `linux_chrome_path` are introduced.
  1. `chrome_flags` is used for setting specific chrome flag.
  1. `mail_dev_notice` field is used for setting mail list who'd like to receive report and the error message when the testing goes into something wrong or receive test report.
  1. If you want to run the test at a specific schedule, you can simply set `useCron` to `true` in `main.js`, then set `sched` const to when you need to schedule the test.
  1. `chromium_builder` field is used for running test automation with specific chromium build from build server and bisect commits automatically. Currently only support for Windows platform. The build server will automatically build chromium at the head of commit id passed by user, then upload chromium build to http://powerbuilder.sh.intel.com/project/chromium_builder/, and then this tool get the corresponding chromium build to run testing at target device.
     1. Single commit:  Once you set `enable_chromium_build` to `true`, you must set the `commit_id` to the specific commit id for building chromium, usually you only need to pass the first 7 characters of a normal chromium commit id. The build server's host and port are set in `host` and `port` fields by default.
     2. Bisect commits: Set `enable_chromium_build` to `true`, `use_bisect` to `true`, `commits` fields to what you commits want to bisect with their commit id and commit number, `pwd` to password of http://powerbuilder.sh.intel.com/project/chromium_builder/.

- Run the test: restart the PC and go to this folder again and run:
  `node main.js`
- Run multiple chrome binaries:
  1. Create `configs` folder under `webpnp-test-automation` directory.
  1. Put config.json files to `configs` folder and change the configuration as you need.
  1. Run `node main.js multi`
- Add a new workload
  If you want to a new workload, you need:
  * Update the `workloads` array of config.json.
  * Provide a workload executor module, place it to `src/workloads/` directory.
  * Update the `executors` object of the function `genWorkloadsResults` in `src/run.js`.

  ## Note

  - This tool uses playwright v1.0.0, which is only guarantee to support Chromium >= 84.0.4135.0.
  - Tester should maintain the cpu_list.json file which is a CPU info lists used for finding matched CPU code name and corresponding competitor's test result.
  - Before testing, please restart the test device to make a clean up environment.
