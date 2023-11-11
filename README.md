# zk_hack_istanbul

## quickstart

all commands mentioned below have been added the [`package.json`](./package.json)'s scripts section, making them easily accessible:

```
yarn run verify
```

or

```
yarn run all
```

## installation

1a. if not installed yet, follow the installation instructions for `rustup`, `circom` and `snarkjs` here: https://docs.circom.io/getting-started/installation/

1b. create the (temp) directories needed for the next steps:

```
mkdir outputs witnesses ceremonies keys verifiers proofs contracts
```

## compilation & proof generation

2. generate the `.wasm` file needed to calculate the witness, and the `.r1cs` file with the circuit's constraints:

```
circom circuit.circom --wasm --r1cs -o outputs/
```

3. calculate the witness using the input from `inputs/circuit.json`:

```
node outputs/circuit_js/generate_witness.js outputs/circuit_js/circuit.wasm inputs/circuit.json witnesses/circuit.wtns
```

4. start ceremony:

```
snarkjs powersoftau new bn128 12 ceremonies/pot12_0000.ptau -v
```

5. contribute to ceremony:

```
snarkjs powersoftau contribute ceremonies/pot12_0000.ptau ceremonies/pot12_0001.ptau --name="First contribution" -v
```

6. start generation of phase 2:

```
snarkjs powersoftau prepare phase2 ceremonies/pot12_0001.ptau ceremonies/pot12_final.ptau -v
```

7. generate `.zkey` file:

```
snarkjs groth16 setup outputs/circuit.r1cs ceremonies/pot12_final.ptau keys/circuit_0000.zkey
```

8. contribute to phase 2:

```
snarkjs zkey contribute keys/circuit_0000.zkey keys/circuit_0001.zkey --name="1st Contributor Name" -v
```

9. export the verification key:

```
snarkjs zkey export verificationkey keys/circuit_0001.zkey verifiers/verification_key.json
```

10. generate proof:

```
snarkjs groth16 prove keys/circuit_0001.zkey witnesses/circuit.wtns proofs/circuit.json proofs/circuit_public.json
```

## verification

11. verify the proof:

```
snarkjs groth16 verify verifiers/verification_key.json proofs/circuit_public.json proofs/circuit.json
```

should show the follwing output:

```
[INFO]  snarkJS: OK!
```

## (optional) solidity

generate a deployable solidity contract that can verify a proof:

```
snarkjs zkey export solidityverifier keys/circuit_0001.zkey contracts/circuit_verifier.sol
```
