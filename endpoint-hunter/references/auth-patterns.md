# Auth Patterns (Lite Summary)

## Pattern A: Wallet Signature (SIWE)
- Standard EIP-4361
- Message: "domain wants you to sign in with your Ethereum address..."
- Nonce from server, sign with private key, POST signature to verify

## Pattern B: OAuth (Google/GitHub)
- PKCE flow recommended
- Generate code_verifier + code_challenge
- User authorize in browser, exchange code for token

## Pattern C: Custom JWT
- POST credentials to /api/auth, receive JWT
- Include in Authorization header, refresh before expiry

## Pattern D: API Key
- Key in header or query param, no expiry

## Detection Tips
- Check Network tab during login
- Look for Authorization header
- Check cookie for session tokens
- Inspect localStorage for tokens

*Full version includes 15+ specific project case studies.*
