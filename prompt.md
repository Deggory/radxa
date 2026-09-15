# HybridPilot Android Auto Integration Prompt

You are working on HybridPilot, running on a Radxa Dragon Q8B 8 GB with Ubuntu 24.04 and Linux kernel 7.0.11-6-qcom.

## Goal

Make the Q8B act as an Android Auto projection device for a 2023 Kia Sonet DCT, so the Kia infotainment display can show the HybridPilot UI/video, output audio through Kia, and send touchscreen/button input back to Q8B.

Target architecture:

```text
Q8B HybridPilot UI/video/audio
        |
     USB-C 0
        |
 Kia Android Auto USB port
        |
Kia display + speakers + touchscreen
```

The Kia display remains the final target. Do not change the goal to using a tablet, generic USB display, generic UVC webcam, generic USB audio, or a web dashboard.

## Completed Work

### Hardware

- Radxa Dragon Q8B, Qualcomm SC8280XP.
- Ubuntu 24.04.5.
- USB-C 0 was configured using `/boot/dtbo/sc8280xp-usb0-peripheral.dtbo`.
- The live device tree confirms `USB0 dr_mode = peripheral`.
- UDC: `/sys/class/udc/a600000.usb`.

### USB Gadget Test

- ConfigFS and `libcomposite` work.
- A temporary CDC ACM serial gadget named `hybridpilot-diag` was created.
- It binds successfully to `a600000.usb`.
- Q8B shows the device-side serial endpoint `/dev/ttyGS0`.
- A Linux PC detects Q8B as `/dev/ttyACM0`.
- Bidirectional USB messages were successfully tested:

```text
PC -> Q8B
Q8B -> PC
```

- When connected to the PC, USB state showed `configured` and `high-speed`.

### Existing Scripts

Repository: <https://github.com/Deggory/radxa>

- `radxa/activate_gadget`
- `radxa/deactivate_gadget`

They create and remove the temporary diagnostic gadget. Do not break these scripts or change persistent USB0 device-tree overlay configuration without a clear reason.

## Known Limitations

- Current `hybridpilot-diag` is only CDC ACM serial.
- It does not provide Android Auto, video, audio, or touch.
- No Android Auto, OpenAuto, AASDK, SNPE, or QNN implementation is installed.
- FunctionFS support is enabled: `CONFIG_USB_CONFIGFS_F_FS=y`.
- Android accessory gadget support is absent: no `f_accessory`, no `android_usb`, and no `CONFIG_USB_CONFIGFS_F_ACC`.
- GStreamer, codecs, protobuf, ADB, FastRPC, and Qualcomm GPU support are present.
- The board has Adreno 690 GPU access through Turnip/Freedreno.
- Do not claim Hexagon NPU support until QNN/SNPE or another compatible runtime is installed and benchmarked.

## Research Task

1. Read and analyze the relevant GitHub projects/references supplied with this request.
2. Determine whether each project implements:
   - Android Auto phone/device side
   - Android Auto head-unit side
   - Android Open Accessory only
   - Generic USB transport only
3. Identify the exact role Q8B must emulate for Kia Android Auto.
4. Explain whether Android Auto can realistically be implemented on this Linux Q8B setup.
5. Identify all required pieces:
   - USB descriptors and USB function choice
   - FunctionFS endpoints
   - Android Auto session negotiation
   - authentication/certification requirements
   - video codec and transport
   - audio codec and transport
   - touchscreen/key input transport
   - display-size negotiation
   - reconnect/error handling
   - systemd startup service
6. Do not confuse Android Open Accessory with Android Auto.
7. Do not claim that generic UVC, USB audio, serial, ADB, RNDIS, or FunctionFS alone makes Android Auto work.

## Implementation Rules

- First produce a factual feasibility report and architecture plan.
- Do not invent undocumented Android Auto packets, descriptors, APIs, or handshake behavior.
- Prefer existing maintained open-source implementations where legally and technically possible.
- If an Android-compatible OS/client stack is required, state this clearly.
- Keep the first code milestone small and testable: replace CDC ACM with a generic FunctionFS bidirectional transport test between Linux PC and Q8B.
- Do not begin Kia testing until FunctionFS transport works reliably with the Linux PC.
- Do not write vehicle CAN commands.
- Do not modify Panda/Chimera safety behavior.
- Do not connect infotainment transport to vehicle-control authority.
- Add logs for USB connection state, negotiated endpoints, reconnects, and failures.
- Every benchmark must report actual measured results.

## Deliverables

1. Feasibility report.
2. Exact protocol-role explanation.
3. Required software/kernel/runtime dependencies.
4. Recommended GitHub project or implementation path.
5. Risks and blockers.
6. Step-by-step milestones from current USB serial success to Kia Android Auto integration.
7. Only then, implement the PC-tested FunctionFS transport milestone.
