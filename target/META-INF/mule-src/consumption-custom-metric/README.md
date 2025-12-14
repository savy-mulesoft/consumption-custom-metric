# Consumption Custom Metric API

A Mule 4 HTTP API that demonstrates custom metrics integration by calling upstream services, measuring payload sizes, and emitting telemetry data.

## API Overview

**Endpoint**: `GET /payloadGen/bytes/{size}`

**Behavior**:
1. Calls upstream service: `GET https://httpbin.org/bytes/{size}` 
2. Computes upstream payload size in bytes
3. Emits custom metrics via Anypoint Custom Metrics Connector
4. Base64-encodes the binary response
5. Returns JSON-wrapped response to caller

## Configuration

### HTTP Listener
- **Host**: 0.0.0.0
- **Port**: 8081
- **Base Path**: /

### HTTP Request (httpbin)
- **Protocol**: HTTPS
- **Host**: httpbin.org
- **Port**: 443

### Custom Metrics
The application emits the `payloadgen_transfer` metric with:

**Dimensions** (low-cardinality):
- `route` = payloadGen_bytes
- `method` = GET
- `status` = 200|4xx|5xx
- `upstream` = httpbin
- `size_bucket` = 1KB|10KB|100KB|1MB|10MB (derived from requested size)

**Facts** (numerical data):
- `requested_bytes` - Size parameter from request
- `upstream_body_bytes` - Actual bytes received from httpbin
- `response_json_bytes` - Size of final JSON response
- `duration_ms` - Total processing time in milliseconds

## Request Validation

- Size must be > 0
- Maximum size: 10,000,000 bytes (10MB)
- Invalid requests return HTTP 400 with error JSON

## Response Format

```json
{
  "requestedSize": 1024,
  "upstreamBodyBytes": 1024,
  "encoding": "base64",
  "data": "<base64-encoded-binary-data>"
}
```

## Dependencies

- **Mule Runtime**: 4.10.0
- **Java**: 17
- **HTTP Connector**: 1.9.3
- **Custom Metrics Extension**: 2.3.2

## Usage Examples

```bash
# Request 1KB of data
curl http://localhost:8081/payloadGen/bytes/1024

# Request 10KB of data  
curl http://localhost:8081/payloadGen/bytes/10240

# Invalid request (too large)
curl http://localhost:8081/payloadGen/bytes/20000000
```

## Build and Run

```bash
# Clean and compile
mvn clean compile

# Package the application
mvn clean package

# Deploy to local Mule runtime
mvn mule:deploy
```

## Architecture

The flow implements the following pattern:
1. **Input Validation** - Parse and validate size parameter
2. **Timer Start** - Record start time for duration calculation
3. **Upstream Call** - HTTP request to httpbin.org
4. **Size Measurement** - Calculate binary payload size
5. **Data Transformation** - Convert to Base64 and wrap in JSON
6. **Metrics Emission** - Send telemetry data with dimensions and facts
7. **Response** - Return JSON to client

## Error Handling

- **VALIDATION:INVALID_SIZE** - Returns HTTP 400 for invalid size parameters
- **Generic Errors** - Returns HTTP 500 for unexpected errors
- All errors include JSON response with error description

## Monitoring

Custom metrics are emitted to Anypoint Monitoring and can be used to:
- Track API usage patterns by size bucket
- Monitor upstream service performance 
- Analyze data transfer volumes
- Alert on error rates or latency issues

## Repository

[GitHub Repository](https://github.com/savy-mulesoft/consumption-custom-metric)
