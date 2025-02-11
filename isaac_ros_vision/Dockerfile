ARG ROS_DISTRO=humble

FROM nvcr.io/nvidia/isaac/ros:x86_64-ros2_humble_e46da4110307fe0d4de2a3060b8e9ba8
ENV ROS_DISTRO=${ROS_DISTRO}
SHELL ["/bin/bash", "-c"]

# Install NVIDIA Toolkit
RUN curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
&& curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list && \ sudo apt-get update && \ sudo apt-get install -y nvidia-container-toolkit

#Configures Docker
RUN sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker

#Restarts Docker
RUN sudo systemctl daemon-reload && sudo systemctl restart docker

#Install Git LFS for large files
RUN sudo apt-get install git-lfs && git lfs install --skip-repo

#Create workspace
RUN mkdir -p  ~/workspaces/isaac_ros-dev/src
ENV ISAAC_ROS_WS=/workspaces/isaac_ros-dev

#Clone isaac_ros_common
RUN cd ${ISAAC_ROS_WS}/src && \
git clone -b release-3.2 https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common.git isaac_ros_common

#Download data from NGC
RUN sudo apt-get install -y curl jq tar \ NGC_ORG="nvidia"
RUN NGC_TEAM="isaac" && \
PACKAGE_NAME="isaac_ros_dope" && \
NGC_RESOURCE="isaac_ros_dope_assets" && \
NGC_FILENAME="quickstart.tar.gz" && \
MAJOR_VERSION=3 && \
MINOR_VERSION=2 && \
VERSION_REQ_URL="https://catalog.ngc.nvidia.com/api/resources/versions?orgName=$NGC_ORG&teamName=$NGC_TEAM&name=$NGC_RESOURCE&isPublic=true&pageNumber=0&pageSize=100&sortOrder=CREATED_DATE_DESC" && \
AVAILABLE_VERSIONS=$(curl -s \ 
    -H "Accept: application/json" "$VERSION_REQ_URL")  \
LATEST_VERSION_ID=$(echo $AVAILABLE_VERSIONS | jq -r "  \
    .recipeVersions[]  \
    | .versionId as \$v  \
    | \$v | select(test(\"^\\\\d+\\\\.\\\\d+\\\\.\\\\d+$\"))  \
    | split(\".\") | {major: .[0]|tonumber, minor: .[1]|tonumber, patch: .[2]|tonumber} \
    | select(.major == $MAJOR_VERSION and .minor <= $MINOR_VERSION) \
    | \$v  \
    " | sort -V | tail -n 1 && \
) \
if [ -z "$LATEST_VERSION_ID" ]; then \
    echo "No corresponding version found for Isaac ROS $MAJOR_VERSION.$MINOR_VERSION" \
    echo "Found versions:" \
    echo $AVAILABLE_VERSIONS | jq -r '.recipeVersions[].versionId' \
else \
    mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets && \
    FILE_REQ_URL="https://api.ngc.nvidia.com/v2/resources/$NGC_ORG/$NGC_TEAM/$NGC_RESOURCE/\
versions/$LATEST_VERSION_ID/files/$NGC_FILENAME" && \
    curl -LO --request GET "${FILE_REQ_URL}" && \
    tar -xf ${NGC_FILENAME} -C ${ISAAC_ROS_WS}/isaac_ros_assets && \
    rm ${NGC_FILENAME} \
fi &&
#Download Ketcup DOPE model for example
RUN mkdir -p ${ISAAC_ROS_WS}/isaac_ros_assets/models/dope/ && \
   mv ~/Downloads/Ketchup.pth ${ISAAC_ROS_WS}/isaac_ros_assets/models/dope

