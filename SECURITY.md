# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in nil.js, please report it by emailing the maintainers or opening a private security advisory on GitHub.

**Please do not open public issues for security vulnerabilities.**

### What to Include

When reporting a vulnerability, please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if available)
- Your contact information

We will acknowledge receipt of your vulnerability report within 48 hours and provide regular updates on our progress.

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 0.21.x  | :white_check_mark: |
| < 0.21  | :x:                |

## Security Best Practices

### Private Key Management

**NEVER** hardcode private keys in your source code or commit them to version control.

#### ✅ GOOD: Use environment variables
```typescript
import { LocalECDSAKeySigner } from '@nilfoundation/niljs';

const signer = new LocalECDSAKeySigner({
  privateKey: process.env.PRIVATE_KEY // Load from environment
});
```

#### ❌ BAD: Hardcoded private key
```typescript
// DON'T DO THIS!
const signer = new LocalECDSAKeySigner({
  privateKey: '0x1234567890abcdef...' // NEVER hardcode keys!
});
```

### Secure Key Storage

#### For Development
- Use `.env` files (add to `.gitignore`)
- Use environment variables
- Consider using key management services

#### For Production
- Use hardware security modules (HSMs)
- Use key management services (AWS KMS, Azure Key Vault, etc.)
- Implement key rotation policies
- Use secure enclaves when available

### Mnemonic Phrase Security

When using mnemonic phrases for key derivation:

```typescript
import { LocalECDSAKeySigner } from '@nilfoundation/niljs';

const signer = new LocalECDSAKeySigner({
  mnemonic: process.env.MNEMONIC // 12 or 24 word phrase
});
```

**Security Guidelines:**
- Store mnemonic phrases with the same security as private keys
- Never log or display mnemonic phrases
- Use BIP39-compliant phrases only
- Consider password-protecting mnemonic phrases

### RPC Endpoint Security

Always use HTTPS for RPC endpoints:

```typescript
import { PublicClient, HttpTransport } from '@nilfoundation/niljs';

// ✅ GOOD: HTTPS endpoint
const client = new PublicClient({
  transport: new HttpTransport({
    endpoint: 'https://rpc.nil.foundation' // HTTPS!
  }),
  shardId: 1
});

// ❌ BAD: HTTP endpoint (vulnerable to MITM attacks)
const client = new PublicClient({
  transport: new HttpTransport({
    endpoint: 'http://rpc.nil.foundation' // Insecure!
  }),
  shardId: 1
});
```

### Transaction Safety

#### Always Verify Recipients
```typescript
// Verify address format before sending
import { isAddress } from '@nilfoundation/niljs';

if (!isAddress(recipientAddress)) {
  throw new Error('Invalid recipient address');
}

await wallet.sendMessage({
  to: recipientAddress,
  value: amount,
  gas: 100000n
});
```

#### Use Bounce Mechanism
Protect against failed transactions by specifying a bounce address:

```typescript
await wallet.sendMessage({
  to: recipientAddress,
  value: amount,
  bounceTo: wallet.address, // Funds returned here if transaction fails
  gas: 100000n
});
```

#### Implement Confirmation Checks
```typescript
const hash = await wallet.sendMessage({ /* ... */ });

// Wait for confirmation
const receipt = await waitTillCompleted(client, hash);

if (receipt.success) {
  console.log('Transaction successful');
} else {
  console.error('Transaction failed:', receipt);
}
```

### Rate Limiting

Implement rate limiting for production applications:

```typescript
// Example rate limiter
class RateLimitedClient {
  private lastRequest = 0;
  private minInterval = 100; // ms between requests

  async request(fn: () => Promise<any>) {
    const now = Date.now();
    const timeSinceLastRequest = now - this.lastRequest;
    
    if (timeSinceLastRequest < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastRequest)
      );
    }
    
    this.lastRequest = Date.now();
    return fn();
  }
}
```

### Input Validation

Always validate user inputs before using them:

```typescript
import { assertIsAddress, assertIsValidShardId } from '@nilfoundation/niljs';

function sendTransaction(to: string, amount: string, shardId: number) {
  // Validate inputs
  assertIsAddress(to);
  assertIsValidShardId(shardId);
  
  const amountBigInt = BigInt(amount);
  if (amountBigInt <= 0n) {
    throw new Error('Amount must be positive');
  }
  
  // Proceed with transaction
}
```

### Gas Estimation

Always estimate gas before sending transactions to avoid failures:

```typescript
const estimatedGas = await client.estimateGas({
  to: recipientAddress,
  from: wallet.address,
  data: callData
}, 'latest');

// Add a buffer (10-20%) for safety
const gasLimit = estimatedGas * 120n / 100n;

await wallet.sendMessage({
  to: recipientAddress,
  value: amount,
  gas: gasLimit
});
```

### Error Handling

Implement comprehensive error handling:

```typescript
import { BaseError, BlockNotFoundError } from '@nilfoundation/niljs';

try {
  await wallet.sendMessage({ /* ... */ });
} catch (error) {
  if (error instanceof BlockNotFoundError) {
    console.error('Block not found:', error.blockNumberOrHash);
  } else if (error instanceof BaseError) {
    console.error('nil.js error:', error.shortMessage);
  } else {
    console.error('Unexpected error:', error);
  }
}
```

### Dependency Security

Keep dependencies up to date:

```bash
# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Update dependencies
npm update
```

### Browser Security

When using nil.js in browsers:

1. **Use Content Security Policy (CSP)**
   ```html
   <meta http-equiv="Content-Security-Policy" 
         content="default-src 'self'; connect-src https://rpc.nil.foundation">
   ```

2. **Avoid Storing Keys in Browser Storage**
   - Don't use localStorage for private keys
   - Consider browser extension wallets
   - Use session-based authentication when possible

3. **Implement Request Signing**
   ```typescript
   // Sign requests to verify they come from your app
   const signature = await signer.sign(requestData);
   ```

### Smart Contract Security

When deploying contracts:

1. **Audit Contract Code**
   - Review bytecode before deployment
   - Use verified contracts when possible
   - Test thoroughly on devnet first

2. **Use Minimal Deployment Salt**
   ```typescript
   // Use deterministic salt for reproducibility
   const salt = BigInt(calculateDeterministicSalt(pubkey));
   
   const wallet = new WalletV1({
     pubkey,
     salt,
     shardId: 1,
     client,
     signer
   });
   ```

3. **Verify Deployment**
   ```typescript
   // Verify contract code after deployment
   const deployedCode = await client.getCode(contractAddress, 'latest');
   
   if (!compareBytes(deployedCode, expectedBytecode)) {
     throw new Error('Deployed code does not match expected bytecode');
   }
   ```

### Network Security

1. **Validate Chain ID**
   ```typescript
   const chainId = await client.chainId();
   
   if (chainId !== EXPECTED_CHAIN_ID) {
     throw new Error('Connected to wrong network');
   }
   ```

2. **Monitor for Reorgs**
   ```typescript
   // Wait for multiple confirmations
   async function waitForConfirmations(hash: Hex, confirmations: number) {
     const receipt = await client.getMessageReceiptByHash(hash);
     if (!receipt) {
       throw new Error('Receipt not found');
     }
     
     const currentBlock = await client.getBlockByNumber('latest', false);
     const confirmationCount = currentBlock.number - receipt.blockNumber;
     
     if (confirmationCount < confirmations) {
       // Wait and check again
       await new Promise(resolve => setTimeout(resolve, 1000));
       return waitForConfirmations(hash, confirmations);
     }
   }
   ```

## Security Checklist

Before deploying to production:

- [ ] Private keys stored securely (HSM, KMS, or secure enclave)
- [ ] No private keys in source code
- [ ] No private keys in version control
- [ ] All RPC endpoints use HTTPS
- [ ] Input validation implemented
- [ ] Error handling implemented
- [ ] Rate limiting implemented
- [ ] Gas estimation with buffer
- [ ] Transaction confirmation checks
- [ ] Bounce addresses configured
- [ ] Chain ID validation
- [ ] Dependencies up to date
- [ ] Security audit completed
- [ ] Test coverage adequate
- [ ] Logging does not expose sensitive data
- [ ] Browser CSP configured (if applicable)

## Known Security Considerations

### 1. Private Key Exposure
The library handles private keys in memory. While they are not logged or exposed, they exist in JavaScript runtime memory and could theoretically be accessed by malicious code running in the same context.

**Mitigation**: Use hardware wallets or secure enclaves for production applications.

### 2. Replay Attacks
The library includes replay protection via sequence numbers (seqno) and chain IDs. However, if seqno is not properly managed, replay attacks could occur.

**Mitigation**: Always use the library's automatic seqno management or carefully track seqno manually.

### 3. Man-in-the-Middle Attacks
If HTTP (non-TLS) endpoints are used, traffic could be intercepted.

**Mitigation**: Always use HTTPS endpoints for production.

### 4. Dependency Vulnerabilities
Third-party dependencies may contain vulnerabilities.

**Mitigation**: Regularly update dependencies and monitor security advisories.

## Security Resources

- [=nil; Security Documentation](https://nil.foundation/security)
- [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- [CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)

## Security Updates

Security updates will be released as patch versions. Subscribe to repository notifications to stay informed about security patches.

## License

This security policy is part of nil.js and is subject to the same MIT license.

---

**Last Updated**: October 17, 2025
