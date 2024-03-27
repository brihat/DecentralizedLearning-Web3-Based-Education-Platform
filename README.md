# DecentralizedLearning-Web3-Based-Education-Platform
DecentralizedLearning is a blockchain-powered platform that provides accessible, affordable, and verifiable education, enabling learners to earn micro-credentials for their skills and knowledge.
from web3 import Web3
from solcx import compile_source
from web3.middleware import geth_poa_middleware

# Solidity source code for the smart contract
contract_source_code = '''
pragma solidity ^0.8.0;

contract DecentralizedLearning {
    struct Course {
        string name;
        string description;
        address educator;
        bool active;
    }

    struct Credential {
        string courseName;
        string studentName;
        string credentialData; // Could include a hash of the certificate details
        bool valid;
    }

    address public platformOwner;
    mapping(uint => Course) public courses;
    mapping(address => Credential[]) public credentials;
    uint public courseCount;

    constructor() {
        platformOwner = msg.sender;
    }

    function createCourse(string memory name, string memory description) public {
        courses[courseCount] = Course(name, description, msg.sender, true);
        courseCount++;
    }

    function issueCredential(address student, string memory courseName, string memory studentName, string memory credentialData) public {
        Credential memory newCredential = Credential(courseName, studentName, credentialData, true);
        credentials[student].push(newCredential);
    }

    function validateCredential(address student, uint credentialIndex) public view returns(bool) {
        return credentials[student][credentialIndex].valid;
    }

    function invalidateCredential(address student, uint credentialIndex) public {
        // Only the educator who issued the credential or the platform owner can invalidate it
        require(msg.sender == credentials[student][credentialIndex].educator || msg.sender == platformOwner, "Unauthorized");
        credentials[student][credentialIndex].valid = false;
    }
}
'''

# Compile the contract
compiled_sol = compile_source(contract_source_code, output_values=["abi", "bin"])
contract_id, contract_interface = compiled_sol.popitem()

# Web3 setup (Assuming you are using Ganache for development purposes)
web3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))
web3.middleware_onion.inject(geth_poa_middleware, layer=0)

# Set up the contract
abi = contract_interface['abi']
bytecode = contract_interface['bin']

# Deploy the contract
DecentralizedLearning = web3.eth.contract(abi=abi, bytecode=bytecode)
tx_hash = DecentralizedLearning.constructor().transact()
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)
contract = web3.eth.contract(
    address=tx_receipt.contractAddress,
    abi=abi,
)

# Example of interaction with the contract
# Note: Replace 'your_account_address' with your actual account address
account_address = 'your_account_address'
web3.eth.default_account = account_address

# Create a course
tx_hash = contract.functions.createCourse("Blockchain Basics", "Introductory course on blockchain technology").transact()
web3.eth.wait_for_transaction_receipt(tx_hash)

# Issue a credential
student_address = 'student_account_address'  # Replace with student's account address
tx_hash = contract.functions.issueCredential(student_address, "Blockchain Basics", "John Doe", "Certificate #123").transact()
web3.eth.wait_for_transaction_receipt(tx_hash)

print("Setup complete. Contract deployed and interactions executed.")
