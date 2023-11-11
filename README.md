# zk_hack_istanbul

## installation

1a. if not installed yet, follow the installation instructions for `rustup`, `circom` and `snarkjs` here: https://docs.circom.io/getting-started/installation/

1b. create the (temp) directories needed for the next steps:

```
mkdir outputs witnesses ceremonies keys verifiers proofs
```

## compilation & proof generation

2. generate the `.wasm` file needed to calculate the witness, and the `.r1cs` file with the circuit's constraints:

```
circom multiplier2.circom --wasm --r1cs -o outputs/
```

3. calculate the witness using the input from `inputs/multiplier2.json`:

```
node outputs/multiplier2_js/generate_witness.js outputs/multiplier2_js/multiplier2.wasm inputs/multiplier2.json witnesses/multiplier2.wtns
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
snarkjs groth16 setup outputs/multiplier2.r1cs ceremonies/pot12_final.ptau keys/multiplier2_0000.zkey
```

8. contribute to phase 2:

```
snarkjs zkey contribute keys/multiplier2_0000.zkey keys/multiplier2_0001.zkey --name="1st Contributor Name" -v
```

9. export the verification key:

```
snarkjs zkey export verificationkey keys/multiplier2_0001.zkey verifiers/verification_key.json
```

10. generate proof:

```
snarkjs groth16 prove keys/multiplier2_0001.zkey witnesses/multiplier2.wtns proofs/multiplier2.json proofs/multiplier2_public.json
```

## verification

11. verify the proof:

```
snarkjs groth16 verify verifiers/verification_key.json proofs/multiplier2_public.json proofs/multiplier2.json
```

should show the follwing output:

```
[INFO]  snarkJS: OK!
```

## (optional) solidity

```
snarkjs zkey export solidityverifier keys/multiplier2_0001.zkey contracts/multiplier2_verifier.sol
```
