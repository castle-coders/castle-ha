parallel(
  "amd64": {
    podTemplate(yaml: """
    apiVersion: v1
    kind: Pod
    metadata:
      name: kaniko
    spec:
      nodeSelector:
        kubernetes.io/arch: amd64
      volumes:
      - name: shared-data
        emptyDir: {}
      containers:
      - name: alpine
        image: alpine
        imagePullPolicy: Always
        command:
        - /bin/cat
        tty: true
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
      - name: kaniko
        image: gcr.io/kaniko-project/executor:v1.9.2-debug
        imagePullPolicy: Always
        command:
        - /busybox/cat
        tty: true
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
        env:
        - name: ARCH
          value: amd64
    """
    ) {
      node(POD_LABEL) {
        stage("checkout") {
          checkout scm
        }

        stage("kaniko") {
          container(name:'kaniko', shell: '/busybox/sh'){
            sh '''#!/busybox/sh
            VERSION=`cat UPSTREAM_VERSION`
            CONTAINER_REGISTRY="docker.local.pw10n.pw"
            CONTAINER_NAME="home-assistant" 
            IMAGE_NAME="$CONTAINER_REGISTRY/$CONTAINER_NAME" 
            IMAGE_VERSION_TAG="$IMAGE_NAME:$VERSION-castle-$ARCH" 
            /kaniko/executor --context `pwd` --destination "$IMAGE_VERSION_TAG" --digest-file=/shared-data/termination-log --build-arg CI_ENV=Jenkins --build-arg GIT_COMMIT=$GIT_COMMIT --build-arg GIT_BRANCH=$GIT_BRANCH --build-arg VERSION=$VERSION --cache=true
            '''
          }
        }
      }
    }
    
  },
  "arm64": {
  podTemplate(yaml: """
    apiVersion: v1
    kind: Pod
    metadata:
      name: kaniko
    spec:
      nodeSelector:
        kubernetes.io/arch: arm64
      volumes:
      - name: shared-data
        emptyDir: {}
      containers:
      - name: alpine
        image: alpine
        imagePullPolicy: Always
        command:
        - /bin/cat
        tty: true
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
      - name: kaniko
        image: gcr.io/kaniko-project/executor:v1.9.2-debug
        imagePullPolicy: Always
        command:
        - /busybox/cat
        tty: true
        volumeMounts:
        - name: shared-data
          mountPath: /shared-data
        env:
        - name: ARCH
          value: arm64 
    """
    ) {
      node(POD_LABEL) {
        stage("checkout") {
          checkout scm
        }

        stage("build and publish image") {
          container(name:'kaniko', shell: '/busybox/sh'){
            sh '''#!/busybox/sh
            VERSION=`cat UPSTREAM_VERSION`
            CONTAINER_REGISTRY="docker.local.pw10n.pw"
            CONTAINER_NAME="home-assistant" 
            IMAGE_NAME="$CONTAINER_REGISTRY/$CONTAINER_NAME" 
            IMAGE_VERSION_TAG="$IMAGE_NAME:$VERSION-castle-$ARCH" 
            /kaniko/executor --context `pwd` --destination "$IMAGE_VERSION_TAG" --digest-file=/shared-data/termination-log --build-arg CI_ENV=Jenkins --build-arg GIT_COMMIT=$GIT_COMMIT --build-arg GIT_BRANCH=$GIT_BRANCH --build-arg VERSION=$VERSION --cache=true
            '''
          }
        }
      }
    }
  }
)

podTemplate(yaml: """
  apiVersion: v1
  kind: Pod
  metadata:
    name: manifest 
  spec:
    containers:
    - name: manifest-tool
      image: mplatform/manifest-tool:alpine-v2.0.8 
      imagePullPolicy: Always
      command:
      - cat
      tty: true
  """
  ) {
    node(POD_LABEL) {
      stage("checkout") {
        checkout scm
      }
      stage("publish manifest"){
        container(name:'manifest-tool', shell:'/bin/sh'){
          sh '''#!/bin/sh
          set -x
          VERSION=`cat UPSTREAM_VERSION`
          CONTAINER_REGISTRY="docker.local.pw10n.pw"
          CONTAINER_NAME="home-assistant" 
          IMAGE_NAME="$CONTAINER_REGISTRY/$CONTAINER_NAME" 
          IMAGE_VERSION_TAG="$IMAGE_NAME:$VERSION-castle" 
          IMAGE_VERSION_TEMPLATE="$IMAGE_NAME:$VERSION-castle-ARCH" 
          manifest-tool push from-args \
            --platforms linux/amd64,linux/arm64 \
            --template $IMAGE_VERSION_TEMPLATE \
            --tags $VERSION,latest \
            --target $IMAGE_VERSION_TAG 
          '''
        }
      }
    }
  }