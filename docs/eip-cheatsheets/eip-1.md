# EIP-1 Cheat Sheet: EIP Purpose and Guidelines

## What Is It?
EIP-1 defines the process for proposing, discussing, and implementing changes to Ethereum. It's the meta-document that governs the EIP process itself.

## Key Points

### EIP Types
- **Standards Track**: Changes affecting most or all Ethereum implementations
  - **Core**: Consensus layer changes requiring a hard fork
  - **Networking**: devp2p protocol, network protocol specs
  - **Interface**: Client API/RPC specifications and standards
  - **ERC**: Application-level standards and conventions
- **Meta**: Process changes or informational documents
- **Informational**: General guidelines or information

### EIP Statuses
- **Idea**: Initial discussion stage
- **Draft**: Formal proposal with complete specification
- **Review**: Active review and consideration
- **Last Call**: Final review period (typically 14 days)
- **Final**: Accepted standard
- **Stagnant**: Inactive proposals (no activity for 6+ months)
- **Withdrawn**: Removed by authors
- **Living**: Continuously updated standards

### EIP Format Requirements
- **Preamble**: Metadata section at the top (title, author, status, etc.)
- **Abstract**: Brief (~200 word) description of the issue
- **Motivation**: Why the existing protocol is inadequate
- **Specification**: Technical details (must be precise and comprehensive)
- **Rationale**: Explanation of design decisions
- **Backwards Compatibility**: Impact on existing implementations
- **Test Cases**: Examples to verify correct implementation (if applicable)
- **Reference Implementation**: Sample code (if applicable)
- **Security Considerations**: Potential security implications
- **Copyright Waiver**: Standard CC0 waiver statement

## EIP Workflow
1. Discuss idea in forums (Ethereum Magicians recommended)
2. Write draft EIP following the template
3. Submit PR to the EIPs repository
4. Address feedback from EIP editors
5. Once merged, EIP is in Draft status
6. Gather community feedback and refine
7. Request Review status from EIP editors
8. Request Last Call status when ready
9. Address Last Call feedback
10. If no major changes needed, moves to Final

## Best Practices
- One EIP per feature/concept
- Be clear and specific in the specification
- Include concrete examples
- Research thoroughly before proposing
- Engage constructively with feedback
- Be patient - the process takes time
- Follow up consistently

## Where to Find
- [EIP-1 on eips.ethereum.org](https://eips.ethereum.org/EIPS/eip-1)
- [EIPs GitHub Repository](https://github.com/ethereum/EIPs)
- [EIP Template](https://github.com/ethereum/EIPs/blob/master/eip-template.md) 