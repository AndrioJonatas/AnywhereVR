# Project WebcamVR Beta Code #
"Soon I will upload the .py files and launch Windows GUI with cx_Freeze or pyinstaller"

- Python - "https://www.python.org/downloads/"
- Python-osc - pip install python-osc
- MediaPipe Pose - pip install mediapipe "https://google.github.io/mediapipe/solutions/pose.html"
- VMT "Manual VirtualMotionTracker" - https://github.com/gpsnmeajp/VirtualMotionTracker


**Python**
_____________________
- main.py (Option 1)
_____________________
    import pose_module as pm


    def main(cam):
    # Change parameter to change input(Camera or videos)
    pm.camera_input(int(cam))


    main(input("Please type your camera id."))
_____________________

- main.py (Option 2)
_____________________

    import pose_module as pm


    def main():
    # Change parameter to change input(Camera or videos)
    pm.camera_input("http://192.168.0.0:8080/video")


    main()
_____________________

**Python-osc**

_____________________

    from pythonosc import udp_client

    "Set OSC client" Title

    client = udp_client.SimpleUDPClient('127.0.0.1', 39570)



    def osc_send2(index, enable, timeoffset, px, py, pz, qx, qy, qz, qw, serial):
    print(f"{index}, {enable}, {timeoffset}, {px}, {py}, {pz}, {qx}, {qy}, {qz}, {qw}, {serial}")
    client.send_message("/VMT/Joint/Driver", [index, enable, timeoffset, px, py, pz, qx, qy, qz, qw, serial]
    
_____________________

**MediaPipe Pose**

    import cv2
    import time
    import mediapipe as mp
    import math_module as mm
    import osc_module as osc


    pose = mp.solutions.pose.Pose(False, 1, True, 0.5, 0.5)
    POSE_LANDMARK = [
    (0, "MPT-Head", (0, 7)),
    (32, "MPT-neck", (32, 0)),
    (11, "MPT-leftShoulder", (11, 13)),
    (12, "MPT-rightShoulder", (12, 14)),
    (13, "MPT-leftElbow", (15, 17)),
    (14, "MPT-rightElbow", (16, 18)),
    (15, "MPT-leftHand", (15, 34)),
    (16, "MPT-rightHand", (16, 35)),
    (33, "MPT-waist", (33, 23)),
    (25, "MPT-leftKnee", (25, 27)),
    (26, "MPT-rightKnee", (26, 28)),
    (27, "MPT-leftFoot", (27, 29)),
    (28, "MPT-rightFoot", (28, 30))
    ]


    def camera_input(input_data):
    p_time = 0
    cap = cv2.VideoCapture(input_data)
    while True:
        success, frame = cap.read()
        if success:
            landmarks = pose_detection(frame)
            if landmarks is not None:
                position_data = get_position_data(landmarks)
                conv_data_openvr(create_virtual_landmark(position_data))
        fps, p_time = fps_count(p_time)
        print("FPS: %d" % fps)
        cv2.imshow("Capture", frame)
        if cv2.waitKey(1) & 0xFF == 27:
            break
    print()
    cap.release()
    cap.destroyAllWindows()


    def pose_detection(frame):
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = pose.process(frame_rgb)
    try:
        landmarks = results.pose_world_landmarks.landmark
        return landmarks
    except:
        pass


    def get_position_data(landmarks):
    position_data = []
    for i in range(0, 32):
        position_data.append([i, landmarks[i].x, landmarks[i].y - 0.8, landmarks[i].z])
    return position_data


    def conv_data_openvr(position_data):
    device_data = []
    for index, device_name, comp_pos in POSE_LANDMARK:
        device_data.append(
            [index, 1, 0., -position_data[index][1], -position_data[index][2], -position_data[index][3], 0., 0., 0., 0.])
        osc.osc_send(index, 1, 0., -position_data[index][1], -position_data[index][2], -position_data[index][3], 0., 0., 0., 0.)
    return device_data


    def create_virtual_landmark(position_data):
    # neck
    x = (position_data[11][1] + position_data[12][1]) / 2
    y = (position_data[11][2] + position_data[12][2]) / 2
    z = (position_data[11][3] + position_data[12][3]) / 2
    position_data.append([32, x, y, z])

    # waist
    x = (position_data[23][1] + position_data[24][1]) / 2
    y = (position_data[23][2] + position_data[24][2]) / 2
    z = (position_data[23][3] + position_data[24][3]) / 2
    position_data.append([33, x, y, z])

    # left hand
    x = (position_data[17][1] + position_data[19][1]) / 2
    y = (position_data[17][2] + position_data[19][2]) / 2
    z = (position_data[17][3] + position_data[19][3]) / 2
    position_data.append([34, x, y, z])

    # Right hand
    x = (position_data[18][1] + position_data[20][1]) / 2
    y = (position_data[18][2] + position_data[20][2]) / 2
    z = (position_data[18][3] + position_data[20][3]) / 2
    position_data.append([35, x, y, z])

    return position_data

    def fps_count(p_time):
    c_time = time.time()
    fps = 1 / (c_time - p_time)
    return int(fps), c_time

**VMT "Manual VirtualMotionTracker"**

https://github.com/gpsnmeajp/VirtualMotionTracker/releases
