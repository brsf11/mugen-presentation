# mugen工具的使用和开发  
## mugen简介  
- mugen是什么（图说明mugen、测试套、测试例的关系）
    - mugen是openEuler社区开放的测试框架，提供公共配置和方法以便社区开发者进行测试代码的编写和执行  
    - mugen提供了  
        - 丰富的openEuler测试代码（用于openEuler x86/AArch64 的测试）  
        - 一套编写测试代码的规范（详见第四部分mugen测试开发）  
        - 用于编写测试的函数库（详见mugen项目的文件结构和第四部分mugen测试开发）  
        - 一个能简化测试执行过程的执行程序（mugen.sh）  
    - mugen结构  
        <img src="../Pic/mugen_testsuite_testcases.png">  
- mugen项目的文件结构（图）及各目录作用 
    - 根目录下脚本  
        - dep_install.sh 用于依赖安装
        - mugen.sh mugen执行脚本  
        - qemu_ctl.sh qemu虚拟机配置脚本，暂未使用  
        - runoet.sh 疑似弃用的执行脚本，功能为mugen.sh的子集  
    - suite2cases 测试套文件，一般和一个服务或软件对应，指定测试用例列表、文件路径和其他信息  
        - 例如clang.json  
            ```json  
            {
                "path": "$OET_PATH/testcases/cli-test/clang",
                "cases": [
                    {
                        "name": "oe_test_clang_01"
                    },
                    {
                        "name": "oe_test_clang_02"
                    },
                    {
                        "name": "oe_test_clang_03"
                    }
                ]
            }
            ```  
    - testcases 测试例代码  
        - cli-test  
        - doc-test  
        - embedded-test  
        - feature-test 特性测试（pkgship 软件包依赖管理工具）  
        - security-test 安全测试  
        - smoke-test 冒烟测试  
        - system-test 系统功能测试  
            - os-basic  
            - os-storage  
        - testsuite 示例  
    - libs/locallibs  
        - mugen本体使用的库  
        - 测试代码用库  
        - 共用库  
        - 库文件和各脚本的依赖关系  
            <img src="../Pic/lib_dependency.png">  
    - doc 测试用例规范，定义了编写测试的要求  
    - License  
## mugen的安装、配置和运行  
- 依赖安装  
    ```shell
    sudo bash dep_install.sh
    ```  
    - python模块下载过慢可以换国内源
    - 本例中使用https://pypi.tuna.tsinghua.edu.cn/simple/  
- 配置  
    - mugen的节点
        - mugen有远程执行测试的功能，每一台机器（虚拟机或物理机），配置为一个节点  
        - 当前QEMU网络配置（用户模式）下远程执行会有问题  
    - 配置节点  
        ```shell
        bash mugen.sh -c --ip 127.0.0.1 --password openEuler12#$ --user root --port 22
        ```  
- 运行测试  
    - 成功一例 clang  
        ```shell  
        [root@openEuler-riscv64 mugen]# bash mugen.sh -f clang
        Mon Jul 11 13:37:31 2022 - INFO  - start to run testcase:oe_test_clang_03.
        Mon Jul 11 13:38:18 2022 - INFO  - The case exit by code 0.
        Mon Jul 11 13:38:19 2022 - INFO  - End to run testcase:oe_test_clang_03.
        Mon Jul 11 13:38:20 2022 - INFO  - start to run testcase:oe_test_clang_01.
        Mon Jul 11 13:38:46 2022 - INFO  - The case exit by code 0.
        Mon Jul 11 13:38:47 2022 - INFO  - End to run testcase:oe_test_clang_01.
        Mon Jul 11 13:38:49 2022 - INFO  - start to run testcase:oe_test_clang_02.
        Mon Jul 11 13:39:13 2022 - INFO  - The case exit by code 0.
        Mon Jul 11 13:39:14 2022 - INFO  - End to run testcase:oe_test_clang_02.
        Mon Jul 11 13:39:15 2022 - INFO  - A total of 3 use cases were executed, with 3 successes and 0 failures.
        ```
    - 失败一例 dnf  
        ```shell  
        [root@openEuler-riscv64 mugen]# bash mugen.sh -f dnf
        ...
        Mon Jul 11 13:46:53 2022 - INFO  - start to run testcase:oe_test_dnf_priority.
        Mon Jul 11 13:47:13 2022 - ERROR - The case exit by code 2.
        Mon Jul 11 13:47:14 2022 - INFO  - End to run testcase:oe_test_dnf_priority.
        Mon Jul 11 13:47:16 2022 - INFO  - start to run testcase:oe_test_dnf_reinstall_repoinfo.
        Mon Jul 11 13:50:39 2022 - ERROR - The case exit by code 4.
        Mon Jul 11 13:50:40 2022 - INFO  - End to run testcase:oe_test_dnf_reinstall_repoinfo.
        Mon Jul 11 13:50:41 2022 - INFO  - start to run testcase:oe_test_dnf_all-repos.
        Mon Jul 11 13:51:29 2022 - ERROR - The case exit by code 6.
        Mon Jul 11 13:51:29 2022 - INFO  - End to run testcase:oe_test_dnf_all-repos.
        ...
        Mon Jul 11 14:12:46 2022 - INFO  - A total of 22 use cases were executed, with 11 successes and 11 failures.
        ```  
        - 日志文件 logs/dnf/oe_test_dnf_all-repos/2022-07-12-00:29:34.log   
        ```  
        Tue Jul 12 00:29:37 2022 - INFO  - Start to run test.
        Error: Unknown repo: 'OS'
        Error: Unknown repo: 'everything'
        Error: Unknown repo: 'EPOL'
        Error: Unknown repo: 'debuginfo'
        ```  
        - openEuler RISC-V yum源配置原因  
        ```  
        # just for test
        [mainline]
        name=mainline
        baseurl=https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing>
        enabled=1
        gpgcheck=0
        # just for test
        [epol]
        name=epol
        baseurl=https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing>
        enabled=1
        gpgcheck=0
        [extra]
        name=extra
        baseurl=https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing>
        enabled=1
        gpgcheck=0
        ```  
    - 失败一例 osc  
        ```shell  
        [root@openEuler-riscv64 mugen]# bash mugen.sh -f osc
        Mon Jul 11 14:32:52 2022 - INFO  - start to run testcase:oe_test_osc_03.
        Mon Jul 11 14:33:16 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:33:17 2022 - INFO  - End to run testcase:oe_test_osc_03.
        Mon Jul 11 14:33:19 2022 - INFO  - start to run testcase:oe_test_osc_05.
        Mon Jul 11 14:33:43 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:33:44 2022 - INFO  - End to run testcase:oe_test_osc_05.
        Mon Jul 11 14:33:46 2022 - INFO  - start to run testcase:oe_test_osc_01.
        Mon Jul 11 14:34:13 2022 - ERROR - The case exit by code 7.
        Mon Jul 11 14:34:14 2022 - INFO  - End to run testcase:oe_test_osc_01.
        Mon Jul 11 14:34:15 2022 - INFO  - start to run testcase:oe_test_osc_02.
        Mon Jul 11 14:34:40 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:34:40 2022 - INFO  - End to run testcase:oe_test_osc_02.
        Mon Jul 11 14:34:42 2022 - INFO  - start to run testcase:oe_test_osc_06.
        Mon Jul 11 14:35:07 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:35:07 2022 - INFO  - End to run testcase:oe_test_osc_06.
        Mon Jul 11 14:35:09 2022 - INFO  - start to run testcase:oe_test_osc_07.
        Mon Jul 11 14:35:33 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:35:34 2022 - INFO  - End to run testcase:oe_test_osc_07.
        Mon Jul 11 14:35:36 2022 - INFO  - start to run testcase:oe_test_osc_build.
        Mon Jul 11 14:36:01 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:36:02 2022 - INFO  - End to run testcase:oe_test_osc_build.
        Mon Jul 11 14:36:04 2022 - INFO  - start to run testcase:oe_test_osc_04.
        Mon Jul 11 14:36:28 2022 - ERROR - The case exit by code 1.
        Mon Jul 11 14:36:29 2022 - INFO  - End to run testcase:oe_test_osc_04.
        Mon Jul 11 14:36:29 2022 - INFO  - A total of 8 use cases were executed, with 0 successes and 8 failures.
        ```
- 辅助测试脚本  
    - 功能  
    - 使用  
        - 一例完整展示 list_test  
            - list_test  
            ```
            vim
            coreutils
            net-tools
            ```  
            - 运行结果  
            ```shell
            [root@openEuler-riscv64 mugen]# python3 runtest.py list_test
            Available test suites num = 387
            total test targets num = 3
            Unavailable test targets:
            vim
            coreutils
            Available test targets:
            net-tools
            Mon Jul 11 22:44:10 2022 - INFO  - start to run testcase:oe_test_service_arp-ethers.
            Mon Jul 11 22:45:02 2022 - INFO  - The case exit by code 0.
            Mon Jul 11 22:45:03 2022 - INFO  - End to run testcase:oe_test_service_arp-ethers.
            Mon Jul 11 22:45:04 2022 - INFO  - A total of 1 use cases were executed, with 1 successes and 0 failures.
            Target net-tools tested 1 cases, failed 0 cases
            ```  
        - 一例展示测试列表比对 list_minimal  
            - 运行结果（由于测试运行时间太长，展示不运行测试）  
            ```shell  
            [root@openEuler-riscv64 mugen]# python3 runtest.py list_minimal 
            Available test suites num = 387
            total test targets num = 54
            Unavailable test targets:
            vim
            coreutils
            systemd-udev
            libssh
            passwd
            wget
            procps-ng
            dnf-plugins-core
            rpm-build
            ...
            Available test targets:
            systemd
            net-tools
            openssh
            NetworkManager
            dnf
            git
            hostname
            osc
            iputils
            cpio
            util-linux
            openslp
            lvm2
            git
            kernel
            ERROR: Targets are not tested!
            ```
## mugen基本测试原理  
- 测试运行流程   
    - mugen.sh函数调用过程  
        <img src="../Pic/mugen_procedure.png">  
    - testcase函数调用过程例1（图）  
        - testcase代码结构  
            ```shell  
            source ${OET_PATH}/libs/locallibs/common_lib.sh #包含函数库
            function config_params() {}                     #需要预加载的数据、参数配置
            function pre_test() {}                          #测试对象、测试需要的工具等安装准备
            function run_test() {}                          #测试点的执行
            function post_test() {}                         #后置处理，恢复测试环境
            main "$@"                                       #运行测试
            ```  
        - 例如  
            ```shell  
            source "../common/common_lib.sh"

            function pre_test() {
                LOG_INFO "Start environmental preparation."
                DNF_INSTALL git-daemon
                LOG_INFO "End of environmental preparation!"
            }

            function run_test() {
                LOG_INFO "Start to run test."
                test_execution git.socket
                systemctl start git.socket
                systemctl reload git.socket 2>&1 | grep "Job type reload is not applicable for unit git.socket"
                CHECK_RESULT $?
                systemctl status git.socket | grep "Active: active"
                CHECK_RESULT $?
                LOG_INFO "End of the test."
            }

            function post_test() {
                LOG_INFO "start environment cleanup."
                DNF_REMOVE
                LOG_INFO "Finish environment cleanup!"
            }

            main "$@"
            ```
    - testcase执行函数调用过程  
        <img src="../Pic/testcase_procedure.png">
## mugen测试的开发  
- 开发测试需要编写的文件  
    - testcase文件  
    - common文件（可选）  
    - suite2cases文件  
- testcase常见测试编写方法  
    - 合理运用mugen的库  
        - 全局common_lib  
            - LOG_INFO/WARN/DEBUG/ERROR() 用于打印信息  
            - CHECK_RESULT() 对比结果  
                - ```CHECK_RESULT $?``` 判断上一语句返回值是否为0，否则判错  
                - ```CHECK_RESULT $? 0 0 "log content"``` 判断上一语句返回值是否为0，否则判错并输出信息log content
            - DNF_INSTALL/REMOVE() 用于安装和卸载需要测试的软件包  
            - SLEEP_WAIT() 等待命令执行时长，超时判错  
        - cli-test中的systemd单元（详见 testcases/cli-test/common/common_lib.sh）   
- 测试代码开发例（GCC） 
## 待完成的工作  
- 开发mugen测试代码  
    - 从mugen自带的测试代码中选出可用于openEuler RISC-V测试的  
    - 找到并编写mugen自带测试代码中缺少的测试  
    - 最终形成一个测试列表  
- mugen的其他功能（如远程执行测试、machine type、machine num等）  
- 完善辅助测试脚本，实现更多功能  