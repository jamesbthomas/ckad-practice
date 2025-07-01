# CKAD Practice
A simple repository designed to practice the elements associated with the Certified Kubernetes Application Developer (CKAD) exam.

## CKAD Reference
CKAD is offered by the Linux Computing Foundation in partnership with the Cloud Native Computing Foundation.

To sign up for the exam: https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/#domains

Helpful Udemy Course: https://obscuritylabs.udemy.com/course/certified-kubernetes-application-developer

## Dependencies and Environment
This repository is designed to work locally, with no external cloud resources and as many free tools as possible.
### Development Environment
- IDE: VSCode
  - Extensions:
    - YAML by RedHat
    - Python by Microsoft
    - Markdown All in One by Yu Zhang
    - Kubernetes by Microsoft
    - Code Spell Checker by Street Side Software
### Practice Environment
- Host OS: Windows (don't hate me, I don't have the patience to run my home environment off Linux)
- Hypervisor: VirtualBox
- Guest OS: 
- Guest VMs:
  - Kubernetes Manager: runs the orchestration services and hosts the container images for the nodes
  - Kubernetes Nodes: uses two nodes to experiment with setting up multi-node networking
    - node1 - first worker node
    - node2 - second worker node