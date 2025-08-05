To setup multiple machines, we recommend using a different RMW. Either go with Zenoh or CycloneDDS.
We prefer to use CycloneDDS.

## Using CycloneDDS for multi-machine setups

1. Make sure that the Cyclone RMW is installed `ros-$ROS_DISTRO-rmw-cyclonedds-cpp`. If you use the `pixi.toml` provided
    in this repo it should be the case.


2. Now you can modify your`scripts/set_env.sh` to include further configuration lines: 
    ```bash hl_lines="4-7" title="scripts/set_env.sh"
    export ROS_DOMAIN_ID=100  # (1)! 
    export CRISP_CONFIG_PATH=/path/to/crisp_py/config  # (1)!

    export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
    export CYCLONEDDS_URI=file:///path/to/crisp_gym/scripts/cyclone_config.xml  # (2)!

    ros2 daemon stop && ros2 daemon start  # (3)!
    ```

    1. Check the [getting started](../getting_started.md) to see why we set this.
    2. __TODO__: this file as well as the path need to be modified!
    3. The communication daemon needs to be restarted to account for the changes.

3. Update the config for your setup to use the correct network interface:
    ```bash hl_lines="6" title="scripts/cyclone_config.xml"
    <CycloneDDS xmlns="https://cdds.io/config">
    <Domain>
        <General>
            <AllowMulticast>true</AllowMulticast>
            <Interfaces>
                <NetworkInterface name="enx607d0937fb24" />   # (1)!
            </Interfaces>
        </General>
        <Discovery>
            <ParticipantIndex>auto</ParticipantIndex>
            <MaxAutoParticipantIndex>100</MaxAutoParticipantIndex>
        </Discovery>
    </Domain>
    </CycloneDDS>

    ```

    1. Modify this with your network interface: check `ip addr` on your shell.


4. Finally, check that everything is working. 
Enter in the humble shell with `pixi shell -e humble` and if your robot is active, run `ros2 topic list` and you should see some topics listed!

## Using Zenoh for multi-machine setups

> Zenoh /zeno/ is a pub/sub/query protocol unifying data in motion, data at rest and computations. 

From [Zenoh's website](https://zenoh.io/).

On the other-hand [rmw_zenoh](https://github.com/ros2/rmw_zenoh) is a ROS middleware to use Zenoh as a the pub/sub communication instead of DDS developed by Intrinsic.
To learn more about Zenoh, check their website and learn abour the Zenoh middleware in their repository.

What is important for us to know about Zenoh is that a router is required for discovery similar to how roscore worked in ROS1.
It can be configured with multicast and the avoid using the router but we will avoid a multicast setup since it might cause problems in some networks for example in universities.

### In the host machine - where controllers run

In the machine running the controllers, make sure that you start one Zenoh router.
In the demos, we provide a service to start the router:
```bash
docker compose up launch_zenoh_router
```

This will start a Zenoh router, then all other can be initialized.
The zenoh versions of the docker services end with `_zenoh` and launch the same nodes with Zenoh instead of the default ROS middleware.


### In the remote machine - where the learning policy runs

In this part, we assume that you already installed the [gym or python interface](../getting_started.md#4-getting-started-with-crisp_gym)

1. Make sure that the Zenoh RMW is installed `ros-$ROS_DISTRO-rmw-zenoh-cpp`. If you use the `pixi.toml` provided 
    in this repo it should be the case.

2. Now you can modify your`scripts/set_env.sh` to include further configuration lines: 
    ```bash hl_lines="4-8" title="scripts/set_env.sh"
    export ROS_DOMAIN_ID=100  # (1)! 
    export CRISP_CONFIG_PATH=/path/to/crisp_py/config  # (1)!

    export RMW_IMPLEMENTATION=rmw_zenoh_cpp
    export ZENOH_CONFIG_OVERRIDE='mode="client";\
      connect/endpoints=["tcp/YOUR_HOST_IP:7447"]'  # (2)!

    ros2 daemon stop && ros2 daemon start  # (3)!
    ```

    1. Check the [getting started](../getting_started.md) to see why we set this.
    2. __TODO__: modify this to use the IP address 
    3. The communication daemon needs to be restarted to account for the changes.

    3. Finally, check that everything is working. 
    Enter in the humble shell with `pixi shell -e humble` and if your robot is active, run `ros2 topic list` and you should see some topics listed!

## Troubleshooting

- Make sure that the Zenoh versions are the same across all the machines!
- Run `env | grep RMW_IMPLEMENTATION`, if the variable is not set, you need to make sure that the script `scripts/set_env.sh` is being executed!
- Be sure that the name of the interface is correct!

## References

- iRobot Middleware Config: [https://iroboteducation.github.io/create3_docs/setup/xml-config/](https://iroboteducation.github.io/create3_docs/setup/xml-config/)
- MoveitPro customize DDS: [https://docs.picknik.ai/how_to/configuration_tutorials/customize_dds_configuration/](https://docs.picknik.ai/how_to/configuration_tutorials/customize_dds_configuration/)

- Cyclone Run-Time-Configuration: [https://github.com/eclipse-cyclonedds/cyclonedds/tree/a10ced3c81cc009e7176912190f710331a4d6caf#run-time-configuration](https://github.com/eclipse-cyclonedds/cyclonedds/tree/a10ced3c81cc009e7176912190f710331a4d6caf#run-time-configuration)
- StereoLabs improve performance: [https://www.stereolabs.com/docs/ros2/dds_and_network_tuning#change-dds-middleware](https://www.stereolabs.com/docs/ros2/dds_and_network_tuning#change-dds-middleware)
- Husarion DDS setup: [https://husarion.com/tutorials/other-tutorials/husarnet-cyclone-dds/](https://husarion.com/tutorials/other-tutorials/husarnet-cyclone-dds/)

- ROS2 Doctor: [https://docs.ros.org/en/kilted/Tutorials/Beginner-Client-Libraries/Getting-Started-With-Ros2doctor.html](https://docs.ros.org/en/kilted/Tutorials/Beginner-Client-Libraries/Getting-Started-With-Ros2doctor.html)
