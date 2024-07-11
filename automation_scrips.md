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
