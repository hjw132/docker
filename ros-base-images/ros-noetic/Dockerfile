#FROM osrf/ros:noetic-desktop-full
FROM osrf/ros:noetic-desktop-full
# 定义用户和用户组
ARG USERNAME=Elaina
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG GROUP_NAME=wheel
ENV DEBIAN_FRONTEND=noninteractive
#安装必要软件
RUN apt-get update \
    && apt-get install -y  net-tools nautilus bash-completion clangd git gedit gdb nano software-properties-common wget curl vim
# nautilus-extension-gnome-terminal dbus-x11 libcanberra-gtk-module libcanberra-gtk3-module git-lfs
# 创建用户组和用户

# 第一阶段：系统设置和用户创建
RUN set -eux; \
    # 清理可能的已有用户/组
    if getent passwd 1000 > /dev/null; then \
    existing_user=$(getent passwd 1000 | cut -d: -f1); \
    userdel -r "$existing_user" 2>/dev/null || true; \
    fi; \
    if getent group 1000 > /dev/null; then \
    existing_group=$(getent group 1000 | cut -d: -f1); \
    if ! getent passwd | awk -F: '{print $1":"$4}' | grep -q ":1000$"; then \
    groupdel "$existing_group" 2>/dev/null || true; \
    fi; \
    fi; \
    # 创建新用户组和用户
    groupadd --gid $USER_GID ${GROUP_NAME} && \
    useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME && \
    echo "$USERNAME:password" | chpasswd && \
    usermod -aG ${GROUP_NAME} $USERNAME && \
    echo "%${GROUP_NAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    chown $USERNAME:$GROUP_NAME /home/$USERNAME/.bashrc
WORKDIR /home/${USERNAME}
USER ${USERNAME}
RUN mkdir packages && cd packages && git clone https://github.com/Livox-SDK/Livox-SDK.git && cd Livox-SDK/build \
    && cmake .. && make -j8 && sudo make install \
    && echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> /home/${USERNAME}/.bashrc 
#安装雷达ros驱动
RUN . /opt/ros/noetic/setup.sh \
    && cd packages && mkdir ws_livox && cd ws_livox && mkdir src && cd src \
    && git clone https://github.com/Livox-SDK/livox_ros_driver.git && cd .. && catkin_make \
    && echo 'source ~/packages/ws_livox/devel/setup.bash' >> /home/Elaina/.bashrc \
    && git config --global core.filemode false