# Forward pod port to local port
```
function kube_forward() {
    while getopts p:n:r: option
    do
    case "${option}"
    in
    p) POD_NAME_PATTERN=${OPTARG};;
    n) NAMESPACE=${OPTARG};;
    r) LOCAL_PORT=${OPTARG};;
    esac
    done

    # Ensure LOCAL_PORT is provided
    if [ -z "$LOCAL_PORT" ]; then
        echo "Local port is required."
        return 1
    fi

    # Get the Pod name
    POD_NAME=$(kubectl get pods -n $NAMESPACE --no-headers | grep -E "$POD_NAME_PATTERN" | awk '/Running/ {print $1}' | head -n 1)

    # Print the Pod name
    echo "Selected Pod: $POD_NAME"
    
    # Check if a pod was found
    if [ -z "$POD_NAME" ]; then
        echo "No running pod found matching pattern $POD_NAME_PATTERN in namespace $NAMESPACE"
        return 1
    fi

    # Get the Pod's port
    POD_PORT=$(kubectl get pod -n $NAMESPACE $POD_NAME -o jsonpath='{.spec.containers[*].ports[0].containerPort}')

    # Print the extracted port
    echo "Extracted Pod Port: $POD_PORT"

    # Check if a port was found
    if [ -z "$POD_PORT" ]; then
        echo "No port found for pod $POD_NAME"
        return 1
    fi

    # Forward the pod to a specific local port
    kubectl port-forward -n $NAMESPACE $POD_NAME $LOCAL_PORT:$POD_PORT
}
```

# Get pod config
```#!/bin/zsh

# Function to display the usage of the script
usage() {
    echo "Usage: $0 --pod <wildcard> --namespace <namespace>"
    echo "   or: $0 -p <wildcard> -n <namespace>"
    exit 1
}

# Parse named arguments
while [[ "$#" -gt 0 ]]; do
    case $1 in
        --pod|-p) wildcard="$2"; shift ;;
        --namespace|-n) namespace="$2"; shift ;;
        *) echo "Unknown parameter passed: $1"; usage ;;
    esac
    shift
done

# Check if both arguments are provided
if [ -z "$wildcard" ] || [ -z "$namespace" ]; then
    usage
fi

# Get the first pod name that matches the wildcard
pod_name=$(kubectl get pods -n $namespace -o custom-columns=:metadata.name | grep $wildcard | head -n 1)

# Check if a pod name was found
if [ -z "$pod_name" ]; then
    echo "No pod found matching wildcard '$wildcard' in namespace '$namespace'"
    exit 1
fi

# Get and output the pod configuration
kubectl get pod $pod_name -n $namespace -o yaml
```
