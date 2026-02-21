# Questions regarding the dSpace system

Need more details about the "SCALEXIO Rack System" mentioned in the PPT. 
- Exact details of SCALEXIO Rack System used in the existing gHiL system.
- Since we are talking about the dSpace system. We need Matlab/Simulink Vehicle models
    - Who will provide the Matlab/Simulink vehicle models?
- How are we going to simulate the environments?
  - Replaying the sensor and/or video data? (What tools are used to replay the data?)
  - Synthetic data simulation from the dspace? (Details about how these data is fed into the ADAS subsystems?)

![Details](https://www.dspace.com/_clickr/cfml/index.cfm?src=../../../../../../../files/jpg27/workflow_toolchain_slx_rack_1200px_03_231211_e.jpg&fallback&width=1920&height=1080)

# Hardware Architecture (Topology)

```mermaid
graph LR
    subgraph Rack [dSPACE SCALEXIO Chassis]
        RT_Unit[Processing Unit Core]
        IOCNET((IOCNET Bus))
        
        subgraph IO_Cards [I/O Tier]
            Eth_Mod[10G Automotive Ethernet]
            CAN_Mod[CAN FD / FlexRay]
            GMSL_Mod[Video Interface / GMSL]
        end
    end

    subgraph DUT [Automotive HPC]
        HPC_Core[Main SoC]
        HPC_Eth[Eth Port]
        HPC_Vid[Video Input]
    end

    RT_Unit <--> IOCNET
    IOCNET <--> IO_Cards
    
    Eth_Mod <==>|SOME/IP / DDS| HPC_Eth
    GMSL_Mod ==>|Raw Video Stream| HPC_Vid
    CAN_Mod <-->|Vehicle Bus Data| HPC_Core

    style Rack fill:#f5f5f5,stroke:#333,stroke-width:2px
    style DUT fill:#e1f5fe,stroke:#01579b,stroke-width:2px
```
[GMSL](https://www.google.com/search?q=GMSL&sourceid=chrome&ie=UTF-8#:~:text=mistakes.%20Learn%20more-,Gigabit%20Multimedia%20Serial%20Link,-Wikipedia)   
[IOCNET (I/O Carrier Network)](https://www.dspace.com/en/pub/home/products/hw/simulator_hardware/scalexio/scalexio_iocnet.cfm)    


# Sensor data injection

```mermaid
graph TD
    subgraph Env_Sim [Environment Simulation]
        ASM[ASM Vehicle Dynamics]
        AURELION[AURELION Scene Generator]
    end

    subgraph Injection_Hardware [dSPACE Hardware]
        GPU_Output[High-End GPU]
        Video_Box[D-PDU / GMSL Serializer]
    end

    subgraph Perception [HPC Logic]
        Vision_Algo[Object Detection Algo]
        Planner[Path Planning]
    end

    ASM -->|Object Coordinates| AURELION
    AURELION -->|Raytracing/Video| GPU_Output
    GPU_Output -->|HDMI/DP| Video_Box
    Video_Box -->|LVDS/GMSL| Vision_Algo
    Vision_Algo -->|Detected Lead Car| Planner
    Planner -->|Brake Command| ASM

    linkStyle 3,4 stroke:#ff0000,stroke-width:2px;
```

# SW Stack and toolchains

```mermaid
graph TD
    subgraph Design_Phase [Model Development]
        SL[MATLAB / Simulink]
        MD[ModelDesk - Parameters]
    end

    subgraph Config_Phase [Hardware Mapping]
        CD_Config[ConfigurationDesk]
        Pin_Map[I/O & Net Topology]
    end

    subgraph Execution_Phase [Real-Time Testing]
        RT_App[Real-Time Application]
        ControlDesk[ControlDesk - Dashboard]
        AutoDesk[AutomationDesk - Python]
    end

    SL --> CD_Config
    MD --> CD_Config
    CD_Config -->|Build/Download| RT_App
    RT_App <--> ControlDesk
    ControlDesk --- AutoDesk
```


# Electrical fault simulation

```mermaid

graph LR
    IO[dSPACE I/O Pin]
    HPC_Pin[HPC Input Pin]
    
    subgraph FIU [Failure Insertion Unit]
        S1{Switch Matrix}
        GND[Ground]
        BATT[Battery VCC]
        OPEN[Open Circuit]
    end

    IO --> S1
    S1 --> HPC_Pin
    
    S1 -.->|Fault Trigger| GND
    S1 -.->|Fault Trigger| BATT
    S1 -.->|Fault Trigger| OPEN

    style FIU fill:#fff3e0,stroke:#ff6f00
```

# Latency and sync

```mermaid
sequenceDiagram
    participant Sim as dSPACE Master Clock
    participant Net as Automotive Ethernet (PTP)
    participant HPC as Automotive HPC (Slave)

    Note over Sim, HPC: Time Synchronization (IEEE 802.1AS)
    Sim->>Net: Sync Message (t1)
    Net->>HPC: Sync Message (t1')
    HPC->>Net: Delay Request
    Net->>Sim: Delay Request
    Sim->>HPC: Correction Factor
    Note right of HPC: Clock Drift Corrected
```

# Run1


```mermaid
graph TD
    subgraph dSPACE_SCALEXIO_Rack [dSPACE SCALEXIO Rack]
        RealTime_Proc[Real-Time Processor Unit]
        I_O_Modules[I/O & Communication Boards]
        Sensor_Sim[Sensor Simulation - AURELION/ASM]
        FIU[Failure Insertion Unit]
    end

    subgraph HPC_DUT [High Computing Processor - DUT]
        SOC[HPC SoC - e.g., NVIDIA/Qualcomm]
        SW_Stack[Middleware: AUTOSAR / ROS / QNX]
    end

    %% Connections
    RealTime_Proc -->|Simulation Data| I_O_Modules
    I_O_Modules -->|CAN FD / Eth / GMSL| FIU
    FIU -->|Injected Signals| HPC_DUT
    HPC_DUT -->|Control Commands| I_O_Modules
    I_O_Modules -->|Feedback| RealTime_Proc

    %% Styling
    style dSPACE_SCALEXIO_Rack fill:#f9f,stroke:#333,stroke-width:2px
    style HPC_DUT fill:#bbf,stroke:#333,stroke-width:2px

```
