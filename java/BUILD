package(default_visibility = ["//visibility:public"])

load("@io_bazel_rules_docker//container:container.bzl", "container_image")
load("@io_bazel_rules_docker//contrib:test.bzl", "container_test")
load("@io_bazel_rules_docker//java:image.bzl", "java_image")
load("@package_bundle//file:packages.bzl", "packages", "versions")
load("@package_bundle_debian10//file:packages.bzl", packages_debian10 = "packages", versions_debian10 = "versions")
load("//cacerts:java.bzl", "cacerts_java")
load("//java:jre_ver.bzl", "jre_ver")

DISTRO_SUFFIXES = ("_debian9", "_debian10")

DISTRO_PACKAGES = {
    "_debian9": packages,
    "_debian10": packages_debian10,
}

# distribution-specific deb dependencies added by StreamSets
DISTRO_DEBS = {
    "_debian9": [
        "libtinfo5",
        "libprocps6",
    ],
    "_debian10": [
        "libtinfo6",
        "libprocps7",
    ],
}

DISTRO_VERSIONS = {
    "_debian9": versions,
    "_debian10": versions_debian10,
}

cacerts_java(
    name = "cacerts_java",
)

[container_image(
    name = rule_name + distro_suffix,
    base = ("//cc:cc" if (not ("debug" in rule_name)) else "//cc:debug") + distro_suffix,
    debs = [
        DISTRO_PACKAGES[distro_suffix]["zlib1g"],
        DISTRO_PACKAGES[distro_suffix]["libjpeg62-turbo"],
        DISTRO_PACKAGES[distro_suffix]["libpng16-16"],
        DISTRO_PACKAGES[distro_suffix]["liblcms2-2"],
        DISTRO_PACKAGES[distro_suffix]["libfreetype6"],
        DISTRO_PACKAGES[distro_suffix]["fonts-dejavu-core"],
        DISTRO_PACKAGES[distro_suffix]["fontconfig-config"],
        DISTRO_PACKAGES[distro_suffix]["libexpat1"],
        DISTRO_PACKAGES[distro_suffix]["libfontconfig1"],
        DISTRO_PACKAGES[distro_suffix][jre_deb],
        # Added by StreamSets
        DISTRO_PACKAGES[distro_suffix]["bash"],
        DISTRO_PACKAGES[distro_suffix]["dash"],
        DISTRO_PACKAGES[distro_suffix]["libc6"],
        DISTRO_PACKAGES[distro_suffix]["debianutils"],
        # Added by StreamSets for Job sidecar workaround
        DISTRO_PACKAGES[distro_suffix]["coreutils"],
        DISTRO_PACKAGES[distro_suffix]["procps"],
        DISTRO_PACKAGES[distro_suffix]["libsystemd0"],
        DISTRO_PACKAGES[distro_suffix]["libselinux1"],
        DISTRO_PACKAGES[distro_suffix]["liblzma5"],
        DISTRO_PACKAGES[distro_suffix]["liblz4-1"],
        DISTRO_PACKAGES[distro_suffix]["libgcrypt20"],
        DISTRO_PACKAGES[distro_suffix]["libpcre3"],
        DISTRO_PACKAGES[distro_suffix]["libgpg-error0"],
        # Added by StreamSets for working with archives
        DISTRO_PACKAGES[distro_suffix]["tar"],
        DISTRO_PACKAGES[distro_suffix]["bzip2"],
        DISTRO_PACKAGES[distro_suffix]["gzip"],
        DISTRO_PACKAGES[distro_suffix]["libacl1"],
        DISTRO_PACKAGES[distro_suffix]["libattr1"],
    ] + [DISTRO_PACKAGES[distro_suffix][deb] for deb in DISTRO_DEBS[distro_suffix]],
    # We expect users to use:
    # cmd = ["/path/to/deploy.jar", "--option1", ...]
    entrypoint = [
        "/usr/bin/java",
        "-jar",
    ],
    env = {
        "JAVA_VERSION": jre_ver(DISTRO_VERSIONS[distro_suffix][jre_deb]),
    },
    symlinks = {
        "/usr/bin/java": java_executable_path,
    },
    tars = [":cacerts_java"],
) for (distro_suffix, rule_name, jre_deb, java_executable_path) in [
    ("_debian9", "java8", "openjdk-8-jre-headless", "/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java"),
    ("_debian9", "java8_debug", "openjdk-8-jre-headless", "/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java"),
    ("_debian9", "java11", "openjdk-11-jre-headless", "/usr/lib/jvm/java-11-openjdk-amd64/bin/java"),
    ("_debian9", "java11_debug", "openjdk-11-jre-headless", "/usr/lib/jvm/java-11-openjdk-amd64/bin/java"),
    ("_debian10", "java11", "openjdk-11-jre-headless", "/usr/lib/jvm/java-11-openjdk-amd64/bin/java"),
    ("_debian10", "java11_debug", "openjdk-11-jre-headless", "/usr/lib/jvm/java-11-openjdk-amd64/bin/java"),
]]

# Provide aliases so the default images use debian9
alias(
    name = "java8",
    actual = ":java8_debian9",
)

alias(
    name = "java8_debug",
    actual = ":java8_debug_debian9",
)

alias(
    name = "java11",
    actual = ":java11_debian9",
)

alias(
    name = "java11_debug",
    actual = ":java11_debug_debian9",
)

container_test(
    name = "java8_debian9_test",
    configs = ["testdata/java8.yaml"],
    image = ":java8_debian9",
)

container_test(
    name = "java8_debug_debian9_test",
    configs = ["testdata/java8_debug.yaml"],
    image = ":java8_debug_debian9",
)

[container_test(
    name = "java11" + distro_suffix + "_test",
    configs = ["testdata/java11.yaml"],
    image = ":java11" + distro_suffix,
) for distro_suffix in DISTRO_SUFFIXES]

[container_test(
    name = "java11_debug" + distro_suffix + "_test",
    configs = ["testdata/java11_debug.yaml"],
    image = ":java11_debug" + distro_suffix,
) for distro_suffix in DISTRO_SUFFIXES]

java_image(
    name = "check_certs_java8_debian9",
    srcs = ["testdata/CheckCerts.java"],
    base = "//java:java8_debian9",
    main_class = "testdata.CheckCerts",
)

[java_image(
    name = "check_certs_java11" + distro_suffix,
    srcs = ["testdata/CheckCerts.java"],
    base = "//java:java11" + distro_suffix,
    main_class = "testdata.CheckCerts",
) for distro_suffix in DISTRO_SUFFIXES]

container_test(
    name = "check_certs_java8_debian9_test",
    configs = ["testdata/java8_certs.yaml"],
    image = ":check_certs_java8_debian9",
)

[container_test(
    name = "check_certs_java11" + distro_suffix + "_test",
    configs = ["testdata/java11_certs" + distro_suffix + ".yaml"],
    image = ":check_certs_java11" + distro_suffix,
) for distro_suffix in DISTRO_SUFFIXES]
