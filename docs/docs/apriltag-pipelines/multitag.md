import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# MultiTag Localization

PhotonVision can combine AprilTag detections from multiple simultaneously observed AprilTags from a particular camera with information about where tags are expected to be located on the field to produce a better estimate of where the camera (and therefore robot) is located on the field. PhotonVision can calculate this multi-target result on your coprocessor, reducing CPU usage on your RoboRio. This result is sent over NetworkTables along with other detected targets as part of the `PhotonPipelineResult` provided by PhotonLib.

:::warning
MultiTag requires an accurate field layout JSON to be uploaded! Differences between this layout and the tags' physical location will drive error in the estimated pose output.
:::

## Enabling MultiTag

Ensure that your camera is calibrated and 3D mode is enabled. Navigate to the Output tab and enable "Do Multi-Target Estimation". This enables MultiTag to use the uploaded field layout JSON to calculate your camera's pose in the field. This 3D transform will be shown as an additional table in the "targets" tab, along with the IDs of AprilTags used to compute this transform.

<img src={require("./images/multitag-ui.png").default} alt="Multitarget enabled and running in the PhotonVision UI" width="600"/>

:::note
By default, enabling multi-target will disable calculating camera-to-target transforms for each observed AprilTag target to increase performance; the X/Y/angle numbers shown in the target table of the UI are instead calculated using the tag's expected location (per the field layout JSON) and the field-to-camera transform calculated using MultiTag. If you additionally want the individual camera-to-target transform calculated using SolvePNP for each target, enable "Always Do Single-Target Estimation".
:::

This multi-target pose estimate can be accessed using PhotonLib. We suggest using \{ref}`the PhotonPoseEstimator class <docs/programming/photonlib/robot-pose-estimator:AprilTags and PhotonPoseEstimator>` with the `MULTI_TAG_PNP_ON_COPROCESSOR` strategy to simplify code, but the transform can be directly accessed using `getMultiTagResult`/`MultiTagResult()` (Java/C++).

<Tabs groupId="lang">
  <TabItem value="java" label="Java">
```java
var result = camera.getLatestResult();
if (result.getMultiTagResult().estimatedPose.isPresent) {
  Transform3d fieldToCamera = result.getMultiTagResult().estimatedPose.best;
}
```
  </TabItem>
  <TabItem value="cpp" label="C++">
```cpp
auto result = camera.GetLatestResult();
if (result.MultiTagResult().result.isPresent) {
  frc::Transform3d fieldToCamera = result.MultiTagResult().result.best;
}
``` 
  </TabItem>
</Tabs>

:::note
The returned field to camera transform is a transform from the fixed field origin to the camera's coordinate system. This does not change based on alliance color, and by convention is on the BLUE ALLIANCE wall.
:::

## Updating the Field Layout

PhotonVision ships by default with the [2024 field layout JSON](https://github.com/wpilibsuite/allwpilib/blob/main/apriltag/src/main/native/resources/edu/wpi/first/apriltag/2024-crescendo.json). The layout can be inspected by navigating to the settings tab and scrolling down to the "AprilTag Field Layout" card, as shown below.

<img src={require("./images/field-layout.png").default} alt="The currently saved field layout in the Photon UI" width="600"/>

An updated field layout can be uploaded by navigating to the "Device Control" card of the Settings tab and clicking "Import Settings". In the pop-up dialog, select the "AprilTag Layout" type and choose an updated layout JSON (in the same format as the WPILib field layout JSON linked above) using the paperclip icon, and select "Import Settings". The AprilTag layout in the "AprilTag Field Layout" card below should be updated to reflect the new layout.

:::note
Currently, there is no way to update this layout using PhotonLib, although this feature is under consideration.
:::