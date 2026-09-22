# Observability

## CloudTrail

CloudTrail was enabled in `us-east-1` to record management-plane API activity. The trail delivered events to an S3 destination, providing an audit record of AWS resource changes and the identity responsible for them.

## VPC Flow Logs

VPC Flow Logs were enabled and delivered to S3 to support network troubleshooting and security investigation. The lab reviewed flow-log fields such as source and destination address, ports, protocol, action, and log status.

### Investigation lesson

A flow record marked `ACCEPT` shows that traffic was accepted at the VPC network layer. It does not, by itself, prove an application accepted the connection or that a system was compromised. The resource scope and associated network interface must be identified before drawing conclusions.

## Outcome

The logging work established separate visibility for AWS API activity (CloudTrail) and network metadata (VPC Flow Logs), creating a foundation for later alerting and incident-response exercises.
