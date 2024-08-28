# Forward pod port to local port
```
#!/bin/zsh

# usage guide
usage() {
    echo "Usage: $0 [OPTIONS] [COMMAND]"
    echo
    echo "Commands:"
    echo "  list                   List all forwarded ports without performing any forwarding"
    echo "  kill                   Kill all forwarded ports"
    echo
    echo "Options:"
    echo "  -p <POD_NAME_PATTERN>  Filter pods by name pattern"
    echo "  -n <NAMESPACE>         Specify the namespace"
    echo "  -r <LOCAL_PORT>        Specify the local port"
    echo "  --help                 Display this help message"
    echo
    exit 1
}


# Path to store forwarded ports information
FORWARD_INFO_FILE="/tmp/kube_forwarded_ports.txt"

# Initialize an associative array to store forwarded ports (in-memory only)
declare -A forwarded_ports

# Load existing forwarded ports from the file, if available
if [ -f "$FORWARD_INFO_FILE" ]; then
    echo "Loading existing forwarded ports from $FORWARD_INFO_FILE..."
    while read -r line; do
        pod_name=$(echo "$line" | cut -d',' -f1)
        forward_info=$(echo "$line" | cut -d',' -f2-)
        forwarded_ports[$pod_name]=$forward_info
    done < "$FORWARD_INFO_FILE"
else
    echo "No existing forwarded ports file found. Starting fresh."
fi

# Function to save forwarded ports to the file
function save_forwarded_ports() {
    : > "$FORWARD_INFO_FILE" # Truncate the file
    for pod in "${(@k)forwarded_ports}"; do
        echo "$pod,${forwarded_ports[$pod]}" >> "$FORWARD_INFO_FILE"
        if [ $? -ne 0 ]; then
            echo "Error writing to file"
            return 1
        fi
    done
}


# Function to forward a port
function forward_port() {
    POD_NAME_PATTERN=$1
    NAMESPACE=$2
    LOCAL_PORT=$3

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

    # Save the port-forwarding information
    forwarded_ports[$POD_NAME]="Local: $LOCAL_PORT -> Pod: $POD_PORT"
    if ! save_forwarded_ports; then
        echo "Failed to save forwarded ports."
        return 1
    fi

    # Forward the pod to a specific local port in the background
    echo "Forwarding from 127.0.0.1:$LOCAL_PORT -> $POD_PORT"
    kubectl port-forward -n $NAMESPACE $POD_NAME $LOCAL_PORT:$POD_PORT &
    PORT_FORWARD_PID=$!

    # Update forwarded_ports with PID
    forwarded_ports[$POD_NAME]+=" (PID: $PORT_FORWARD_PID)"
    if ! save_forwarded_ports; then
        echo "Failed to update forwarded ports with PID."
        return 1
    fi

    # Print the forwarded port details
    echo "Port forwarding started: Local: $LOCAL_PORT -> Pod: $POD_PORT (PID: $PORT_FORWARD_PID)"
}

# Function to list all currently active forwarded ports
function list_forwarded_ports() {
    if [ ${#forwarded_ports[@]} -eq 0 ]; then
        echo "No ports are currently forwarded."
        return
    fi

    local active_ports=()
    local stale_ports=()
    
    for pod in "${(@k)forwarded_ports}"; do
        forward_info=${forwarded_ports[$pod]}
        
        # Extract PID from forwarded_ports info
        if [[ "$forward_info" =~ \(PID:\ ([0-9]+)\) ]]; then
            pid="${match[1]}"
            [[ $pid =~ ([0-9]+) ]]
            pid="${match[1]}"
            
            # Check if the PID is still running
           if ps -p $pid > /dev/null;then
                # PID is active, list it
                active_ports+=("$pod: ${forwarded_ports[$pod]}")
            else
                # PID is not active, mark it as stale
                stale_ports+=("$pod")
            fi
       fi
    done

    # Clean up stale ports from the file and memory
    if [ ${#stale_ports[@]} -gt 0 ]; then
        echo "Removing stale entries:"
        for pod in "${stale_ports[@]}"; do
            echo "Removing stale entry for Pod: $pod"
            unset "forwarded_ports[$pod]"
        done

        
        # Rewrite the file to remove stale entries
        if ! save_forwarded_ports; then
            echo "Failed to update forwarded ports after cleaning stale entries."
            return 1
        fi
    fi

    # Output active ports
    if [ ${#active_ports[@]} -eq 0 ]; then
        echo "No active forwarded ports."
     else
        echo "Active ports:"
        for entry in "${active_ports[@]}"; do
            echo "$entry"
        done
    fi
    echo "Stale ports:"
    for entry in "${stale_ports[@]}"; do
        echo "$entry"
    done
}

# Function to kill all forwarded ports
function kill_forwarded_ports() {
    echo "Killing all forwarded ports..."
    for pod in "${(@k)forwarded_ports}"; do
        forward_info=${forwarded_ports[$pod]}
        # Extract PID using parameter expansion and pattern matching
        if [[ "$forward_info" =~ \(PID:\ ([0-9]+)\) ]]; then
            pid="${match[1]}"
            [[ $pid =~ ([0-9]+) ]]
            pid="${match[1]}"
            echo "Killing port-forwarding for Pod: $pod (PID: $pid)"
            if kill "$pid" 2>/dev/null; then
                echo "Successfully killed PID $pid."
                unset "forwarded_ports[$pod]"
            else
                echo "Failed to kill PID $pid. PID may not be valid or process not found."
            fi
        else
            echo "No PID found for Pod: $pod"
        fi
    done
    if ! save_forwarded_ports; then
        echo "Failed to update forwarded ports after killing."
        return 1
    fi
    echo "All forwarded ports have been killed."
}


# Main script logic
# Check if --help is one of the first arguments
if [[ "$1" == "--help" ]]; then
    usage
fi
if [[ "$1" == "list" ]]; then
    # List all forwarded ports without performing any forwarding
    list_forwarded_ports
elif [[ "$1" == "kill" ]]; then
    # Kill all forwarded ports
    kill_forwarded_ports
else
    # Parse command-line arguments
    while getopts p:n:r: option; do
        case "${option}" in
            p) POD_NAME_PATTERN=${OPTARG};;
            n) NAMESPACE=${OPTARG};;
            r) LOCAL_PORT=${OPTARG};;
            *) echo "Usage: $0 -p <POD_NAME_PATTERN> -n <NAMESPACE> -r <LOCAL_PORT>"; exit 1;;
        esac
    done

    # Ensure necessary arguments are provided
    if [[ -z "$POD_NAME_PATTERN" || -z "$NAMESPACE" || -z "$LOCAL_PORT" ]]; then
        usage
        exit 1
    fi

shift $((OPTIND - 1))
if [[ "$#" -ne 0 ]]; then
    usage
fi

    # Forward the pod's port
    forward_port "$POD_NAME_PATTERN" "$NAMESPACE" "$LOCAL_PORT"
fi

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
