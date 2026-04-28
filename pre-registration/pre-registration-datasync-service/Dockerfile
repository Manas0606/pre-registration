# ✅ Use Eclipse Temurin instead of removed openjdk image
FROM eclipse-temurin:11-jdk

ARG SOURCE
ARG COMMIT_HASH
ARG COMMIT_ID
ARG BUILD_TIME

LABEL source=${SOURCE}
LABEL commit_hash=${COMMIT_HASH}
LABEL commit_id=${COMMIT_ID}
LABEL build_time=${BUILD_TIME}

# Build args
ARG spring_config_label
ARG active_profile
ARG spring_config_url
ARG is_glowroot
ARG artifactory_url
ARG iam_adapter_url

# Runtime env variables
ENV active_profile_env=${active_profile}
ENV spring_config_label_env=${spring_config_label}
ENV spring_config_url_env=${spring_config_url}
ENV is_glowroot_env=${is_glowroot}
ENV artifactory_url_env=${artifactory_url}
ENV iam_adapter_url_env=${iam_adapter_url}

# User configs
ARG container_user=mosip
ARG container_user_group=mosip
ARG container_user_uid=1001
ARG container_user_gid=1001

# Install dependencies & create user
RUN apt-get update && \
    apt-get install -y unzip wget && \
    groupadd -g ${container_user_gid} ${container_user_group} && \
    useradd -u ${container_user_uid} -g ${container_user_group} -s /bin/sh -m ${container_user}

# Set working directory
WORKDIR /home/${container_user}

ENV work_dir=/home/${container_user}
ARG loader_path=${work_dir}/additional_jars/
RUN mkdir -p ${loader_path}
ENV loader_path_env=${loader_path}

# ✅ Copy jar (ensure jar exists in target folder)
COPY ./target/pre-registration-datasync-service-*.jar pre-registration-datasync-service.jar

# Permissions
RUN chown -R ${container_user}:${container_user} /home/${container_user}

# Switch user
USER ${container_user_uid}:${container_user_gid}

EXPOSE 9094

# Run app
CMD if [ "$is_glowroot_env" = "present" ]; then \
    wget -q --show-progress "${artifactory_url_env}"/artifactory/libs-release-local/io/mosip/testing/glowroot.zip && \
    wget -q --show-progress "${iam_adapter_url_env}" -O "${loader_path_env}"/kernel-auth-adapter.jar && \
    unzip glowroot.zip && \
    rm -rf glowroot.zip && \
    sed -i 's/<service_name>/pre-registration-datasync-service/g' glowroot/glowroot.properties && \
    java -jar -javaagent:glowroot/glowroot.jar \
    -Dloader.path="${loader_path_env}" \
    -Dspring.cloud.config.label="${spring_config_label_env}" \
    -Dspring.profiles.active="${active_profile_env}" \
    -Dspring.cloud.config.uri="${spring_config_url_env}" \
    pre-registration-datasync-service.jar ; \
    else \
    wget -q --show-progress "${iam_adapter_url_env}" -O "${loader_path_env}"/kernel-auth-adapter.jar && \
    java -jar \
    -Dloader.path="${loader_path_env}" \
    -Dspring.cloud.config.label="${spring_config_label_env}" \
    -Dspring.profiles.active="${active_profile_env}" \
    -Dspring.cloud.config.uri="${spring_config_url_env}" \
    pre-registration-datasync-service.jar ; \
    fi
