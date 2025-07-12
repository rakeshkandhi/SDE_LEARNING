# Security in System Design 🔒

## Overview
Security in system design involves protecting systems, data, and users from threats through comprehensive security measures implemented at every layer of the architecture. It's not an afterthought but a fundamental aspect that must be considered from the beginning.

## Table of Contents
- [Security Principles](#security-principles)
- [Authentication Systems](#authentication-systems)
- [Authorization Models](#authorization-models)
- [Data Protection](#data-protection)
- [Network Security](#network-security)
- [Application Security](#application-security)
- [Infrastructure Security](#infrastructure-security)
- [Security Monitoring](#security-monitoring)
- [Compliance and Governance](#compliance-and-governance)
- [Incident Response](#incident-response)
- [Best Practices](#best-practices)

## Security Principles

### 1. Defense in Depth
Multiple layers of security controls throughout the system.

```
Internet → WAF → Load Balancer → API Gateway → Application → Database
    ↓        ↓         ↓            ↓            ↓           ↓
  DDoS    Firewall   SSL/TLS    Authentication  Input     Encryption
Protection Rules   Termination   Authorization Validation  at Rest
```

### 2. Principle of Least Privilege
Users and systems should have only the minimum access necessary.

```python
# Role-based access control example
class Permission:
    def __init__(self, resource, action):
        self.resource = resource
        self.action = action

class Role:
    def __init__(self, name, permissions):
        self.name = name
        self.permissions = permissions

    def can_perform(self, resource, action):
        return Permission(resource, action) in self.permissions

# Define roles with minimal permissions
read_only_role = Role("read_only", [
    Permission("users", "read"),
    Permission("orders", "read")
])

admin_role = Role("admin", [
    Permission("users", "read"),
    Permission("users", "write"),
    Permission("users", "delete"),
    Permission("orders", "read"),
    Permission("orders", "write"),
    Permission("system", "configure")
])
```

### 3. Fail Securely
When systems fail, they should fail to a secure state.

```python
class SecureService:
    def __init__(self):
        self.security_enabled = True

    def process_request(self, request):
        try:
            # Verify security context
            if not self.verify_security_context(request):
                raise SecurityException("Invalid security context")

            # Process request
            return self.handle_request(request)

        except Exception as e:
            # Fail securely - deny access by default
            self.log_security_event(f"Request failed: {e}")
            raise SecurityException("Access denied")

    def verify_security_context(self, request):
        # If security check fails, return False (secure default)
        try:
            return self.validate_token(request.token)
        except:
            return False  # Fail securely
```

### 4. Zero Trust Architecture
Never trust, always verify - regardless of location or user.

```python
class ZeroTrustGateway:
    def __init__(self):
        self.policy_engine = PolicyEngine()
        self.identity_verifier = IdentityVerifier()
        self.device_verifier = DeviceVerifier()

    def authorize_request(self, request):
        # Verify identity
        identity = self.identity_verifier.verify(request.credentials)
        if not identity:
            return False

        # Verify device
        device_trust = self.device_verifier.verify(request.device_info)
        if device_trust < 0.8:  # Require high device trust
            return False

        # Check policies
        return self.policy_engine.evaluate(
            identity=identity,
            resource=request.resource,
            action=request.action,
            context=request.context
        )
```

## Authentication Systems

### 1. Multi-Factor Authentication (MFA)
```python
import pyotp
import qrcode
from datetime import datetime, timedelta

class MFAService:
    def __init__(self):
        self.backup_codes = {}

    def setup_totp(self, user_id, user_email):
        # Generate secret key
        secret = pyotp.random_base32()

        # Create TOTP object
        totp = pyotp.TOTP(secret)

        # Generate QR code for authenticator app
        provisioning_uri = totp.provisioning_uri(
            name=user_email,
            issuer_name="MyApp"
        )

        qr = qrcode.QRCode()
        qr.add_data(provisioning_uri)
        qr.make()

        # Store secret for user
        self.store_user_secret(user_id, secret)

        return {
            'secret': secret,
            'qr_code': qr.make_image(),
            'backup_codes': self.generate_backup_codes(user_id)
        }

    def verify_totp(self, user_id, token):
        secret = self.get_user_secret(user_id)
        if not secret:
            return False

        totp = pyotp.TOTP(secret)
        return totp.verify(token, valid_window=1)  # Allow 30-second window

    def generate_backup_codes(self, user_id):
        codes = [secrets.token_hex(4) for _ in range(10)]
        self.backup_codes[user_id] = codes
        return codes

    def verify_backup_code(self, user_id, code):
        if user_id in self.backup_codes and code in self.backup_codes[user_id]:
            self.backup_codes[user_id].remove(code)  # Single use
            return True
        return False
```

### 2. OAuth 2.0 / OpenID Connect
```python
# OAuth 2.0 Authorization Server
from flask import Flask, request, jsonify, redirect
import jwt
import secrets

class OAuthServer:
    def __init__(self):
        self.clients = {}  # Registered OAuth clients
        self.authorization_codes = {}
        self.access_tokens = {}
        self.refresh_tokens = {}

    def register_client(self, client_name, redirect_uris):
        client_id = secrets.token_urlsafe(32)
        client_secret = secrets.token_urlsafe(64)

        self.clients[client_id] = {
            'name': client_name,
            'secret': client_secret,
            'redirect_uris': redirect_uris
        }

        return client_id, client_secret

    def authorize(self, client_id, redirect_uri, scope, state):
        # Validate client and redirect URI
        if not self.validate_client(client_id, redirect_uri):
            return {'error': 'invalid_client'}

        # Generate authorization code
        auth_code = secrets.token_urlsafe(32)
        self.authorization_codes[auth_code] = {
            'client_id': client_id,
            'redirect_uri': redirect_uri,
            'scope': scope,
            'expires_at': datetime.now() + timedelta(minutes=10)
        }

        # Redirect to client with authorization code
        return redirect(f"{redirect_uri}?code={auth_code}&state={state}")

    def exchange_code_for_token(self, client_id, client_secret, code, redirect_uri):
        # Validate client credentials
        if not self.validate_client_credentials(client_id, client_secret):
            return {'error': 'invalid_client'}

        # Validate authorization code
        if code not in self.authorization_codes:
            return {'error': 'invalid_grant'}

        auth_data = self.authorization_codes[code]
        if auth_data['client_id'] != client_id or auth_data['redirect_uri'] != redirect_uri:
            return {'error': 'invalid_grant'}

        if datetime.now() > auth_data['expires_at']:
            return {'error': 'expired_grant'}

        # Generate tokens
        access_token = self.generate_access_token(client_id, auth_data['scope'])
        refresh_token = self.generate_refresh_token(client_id)

        # Clean up authorization code
        del self.authorization_codes[code]

        return {
            'access_token': access_token,
            'refresh_token': refresh_token,
            'token_type': 'Bearer',
            'expires_in': 3600
        }
```

### 3. JWT (JSON Web Tokens)
```python
import jwt
import datetime
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa

class JWTService:
    def __init__(self):
        # Generate RSA key pair for signing
        self.private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048
        )
        self.public_key = self.private_key.public_key()

    def generate_token(self, user_id, roles, expires_in_hours=1):
        payload = {
            'user_id': user_id,
            'roles': roles,
            'iat': datetime.datetime.utcnow(),
            'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=expires_in_hours),
            'iss': 'myapp.com',
            'aud': 'myapp-api'
        }

        # Sign with private key
        token = jwt.encode(
            payload,
            self.private_key,
            algorithm='RS256'
        )

        return token

    def verify_token(self, token):
        try:
            payload = jwt.decode(
                token,
                self.public_key,
                algorithms=['RS256'],
                audience='myapp-api',
                issuer='myapp.com'
            )
            return payload
        except jwt.ExpiredSignatureError:
            raise AuthenticationError("Token has expired")
        except jwt.InvalidTokenError:
            raise AuthenticationError("Invalid token")

    def get_public_key_jwks(self):
        """Return public key in JWKS format for token verification"""
        public_key_pem = self.public_key.public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo
        )

        # Convert to JWKS format
        return {
            "keys": [{
                "kty": "RSA",
                "use": "sig",
                "alg": "RS256",
                "n": "...",  # Base64url-encoded modulus
                "e": "AQAB"  # Base64url-encoded exponent
            }]
        }
```

## Authorization Models

### 1. Role-Based Access Control (RBAC)
```python
class RBACSystem:
    def __init__(self):
        self.users = {}
        self.roles = {}
        self.permissions = {}
        self.user_roles = {}
        self.role_permissions = {}

    def create_permission(self, name, resource, action):
        self.permissions[name] = {
            'resource': resource,
            'action': action
        }

    def create_role(self, name, description):
        self.roles[name] = {
            'description': description
        }

    def assign_permission_to_role(self, role_name, permission_name):
        if role_name not in self.role_permissions:
            self.role_permissions[role_name] = []
        self.role_permissions[role_name].append(permission_name)

    def assign_role_to_user(self, user_id, role_name):
        if user_id not in self.user_roles:
            self.user_roles[user_id] = []
        self.user_roles[user_id].append(role_name)

    def check_permission(self, user_id, resource, action):
        # Get user roles
        user_roles = self.user_roles.get(user_id, [])

        # Check each role's permissions
        for role in user_roles:
            role_perms = self.role_permissions.get(role, [])
            for perm_name in role_perms:
                perm = self.permissions.get(perm_name)
                if perm and perm['resource'] == resource and perm['action'] == action:
                    return True

        return False

# Usage example
rbac = RBACSystem()

# Create permissions
rbac.create_permission('read_users', 'users', 'read')
rbac.create_permission('write_users', 'users', 'write')
rbac.create_permission('delete_users', 'users', 'delete')

# Create roles
rbac.create_role('viewer', 'Can view data')
rbac.create_role('editor', 'Can view and edit data')
rbac.create_role('admin', 'Full access')

# Assign permissions to roles
rbac.assign_permission_to_role('viewer', 'read_users')
rbac.assign_permission_to_role('editor', 'read_users')
rbac.assign_permission_to_role('editor', 'write_users')
rbac.assign_permission_to_role('admin', 'read_users')
rbac.assign_permission_to_role('admin', 'write_users')
rbac.assign_permission_to_role('admin', 'delete_users')

# Assign roles to users
rbac.assign_role_to_user('user123', 'editor')

# Check permissions
can_read = rbac.check_permission('user123', 'users', 'read')  # True
can_delete = rbac.check_permission('user123', 'users', 'delete')  # False
```

### 2. Attribute-Based Access Control (ABAC)
```python
class ABACSystem:
    def __init__(self):
        self.policies = []

    def add_policy(self, policy):
        self.policies.append(policy)

    def evaluate(self, subject, resource, action, environment):
        for policy in self.policies:
            if policy.applies(subject, resource, action, environment):
                return policy.evaluate(subject, resource, action, environment)

        return False  # Deny by default

class Policy:
    def __init__(self, name, conditions, effect):
        self.name = name
        self.conditions = conditions
        self.effect = effect  # 'allow' or 'deny'

    def applies(self, subject, resource, action, environment):
        # Check if policy applies to this request
        return True

    def evaluate(self, subject, resource, action, environment):
        # Evaluate all conditions
        for condition in self.conditions:
            if not condition.evaluate(subject, resource, action, environment):
                return False

        return self.effect == 'allow'

class Condition:
    def __init__(self, attribute, operator, value):
        self.attribute = attribute
        self.operator = operator
        self.value = value

    def evaluate(self, subject, resource, action, environment):
        actual_value = self.get_attribute_value(self.attribute, subject, resource, action, environment)

        if self.operator == 'equals':
            return actual_value == self.value
        elif self.operator == 'in':
            return actual_value in self.value
        elif self.operator == 'greater_than':
            return actual_value > self.value

        return False

# Example ABAC policy
time_based_policy = Policy(
    name="business_hours_access",
    conditions=[
        Condition("environment.time", "greater_than", "09:00"),
        Condition("environment.time", "less_than", "17:00"),
        Condition("subject.department", "equals", "finance"),
        Condition("resource.type", "equals", "financial_data")
    ],
    effect="allow"
)
```

## Data Protection

### 1. Encryption at Rest
```python
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import os

class DataEncryption:
    def __init__(self, password=None):
        if password:
            self.key = self.derive_key_from_password(password)
        else:
            self.key = Fernet.generate_key()
        self.cipher = Fernet(self.key)

    def derive_key_from_password(self, password):
        salt = os.urandom(16)
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=100000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
        return key

    def encrypt(self, data):
        if isinstance(data, str):
            data = data.encode()
        return self.cipher.encrypt(data)

    def decrypt(self, encrypted_data):
        return self.cipher.decrypt(encrypted_data).decode()

# Database field encryption
class EncryptedField:
    def __init__(self, encryption_service):
        self.encryption_service = encryption_service

    def encrypt_value(self, value):
        if value is None:
            return None
        return self.encryption_service.encrypt(str(value))

    def decrypt_value(self, encrypted_value):
        if encrypted_value is None:
            return None
        return self.encryption_service.decrypt(encrypted_value)

# Usage in database models
class User:
    def __init__(self, name, email, ssn):
        self.name = name
        self.email = email
        self.encrypted_ssn = encrypted_field.encrypt_value(ssn)

    @property
    def ssn(self):
        return encrypted_field.decrypt_value(self.encrypted_ssn)
```

### 2. Encryption in Transit
```python
# TLS/SSL configuration
import ssl
import socket

class SecureServer:
    def __init__(self, cert_file, key_file):
        self.cert_file = cert_file
        self.key_file = key_file

    def create_secure_context(self):
        context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
        context.load_cert_chain(self.cert_file, self.key_file)

        # Security configurations
        context.minimum_version = ssl.TLSVersion.TLSv1_2
        context.set_ciphers('ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS')
        context.check_hostname = False
        context.verify_mode = ssl.CERT_REQUIRED

        return context

    def start_server(self, host, port):
        context = self.create_secure_context()

        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            sock.bind((host, port))
            sock.listen(5)

            with context.wrap_socket(sock, server_side=True) as ssock:
                while True:
                    conn, addr = ssock.accept()
                    self.handle_client(conn)
```

### 3. Key Management
```python
import boto3
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

class KeyManagementService:
    def __init__(self):
        self.kms_client = boto3.client('kms')
        self.key_cache = {}

    def create_data_key(self, key_id):
        """Create a new data encryption key"""
        response = self.kms_client.generate_data_key(
            KeyId=key_id,
            KeySpec='AES_256'
        )

        return {
            'plaintext_key': response['Plaintext'],
            'encrypted_key': response['CiphertextBlob']
        }

    def decrypt_data_key(self, encrypted_key):
        """Decrypt a data encryption key"""
        response = self.kms_client.decrypt(
            CiphertextBlob=encrypted_key
        )
        return response['Plaintext']

    def encrypt_data(self, data, key_id):
        """Encrypt data using envelope encryption"""
        # Generate data key
        data_key = self.create_data_key(key_id)

        # Encrypt data with data key
        iv = os.urandom(16)
        cipher = Cipher(
            algorithms.AES(data_key['plaintext_key']),
            modes.CBC(iv)
        )
        encryptor = cipher.encryptor()

        # Pad data to block size
        padded_data = self.pad_data(data)
        encrypted_data = encryptor.update(padded_data) + encryptor.finalize()

        return {
            'encrypted_data': encrypted_data,
            'encrypted_key': data_key['encrypted_key'],
            'iv': iv
        }

    def decrypt_data(self, encrypted_package):
        """Decrypt data using envelope encryption"""
        # Decrypt data key
        plaintext_key = self.decrypt_data_key(encrypted_package['encrypted_key'])

        # Decrypt data
        cipher = Cipher(
            algorithms.AES(plaintext_key),
            modes.CBC(encrypted_package['iv'])
        )
        decryptor = cipher.decryptor()

        padded_data = decryptor.update(encrypted_package['encrypted_data']) + decryptor.finalize()
        return self.unpad_data(padded_data)
```

## Network Security

### 1. Web Application Firewall (WAF)
```python
import re
from flask import request, abort

class WAF:
    def __init__(self):
        self.sql_injection_patterns = [
            r"(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER)\b)",
            r"(\b(UNION|OR|AND)\b.*\b(SELECT|INSERT|UPDATE|DELETE)\b)",
            r"('|(\\x27)|(\\x2D)|(\\x2D)|(\\x2D))",
        ]

        self.xss_patterns = [
            r"<script[^>]*>.*?</script>",
            r"javascript:",
            r"on\w+\s*=",
            r"<iframe[^>]*>.*?</iframe>",
        ]

        self.rate_limits = {}

    def check_sql_injection(self, input_string):
        for pattern in self.sql_injection_patterns:
            if re.search(pattern, input_string, re.IGNORECASE):
                return True
        return False

    def check_xss(self, input_string):
        for pattern in self.xss_patterns:
            if re.search(pattern, input_string, re.IGNORECASE):
                return True
        return False

    def check_rate_limit(self, client_ip, limit=100, window=3600):
        current_time = time.time()

        if client_ip not in self.rate_limits:
            self.rate_limits[client_ip] = []

        # Remove old requests outside the window
        self.rate_limits[client_ip] = [
            req_time for req_time in self.rate_limits[client_ip]
            if current_time - req_time < window
        ]

        # Check if limit exceeded
        if len(self.rate_limits[client_ip]) >= limit:
            return False

        # Add current request
        self.rate_limits[client_ip].append(current_time)
        return True

    def validate_request(self):
        client_ip = request.remote_addr

        # Rate limiting
        if not self.check_rate_limit(client_ip):
            abort(429, "Rate limit exceeded")

        # Check all input parameters
        for param_name, param_value in request.values.items():
            if self.check_sql_injection(param_value):
                abort(400, f"SQL injection detected in {param_name}")

            if self.check_xss(param_value):
                abort(400, f"XSS attempt detected in {param_name}")

        # Check request body
        if request.data:
            body = request.data.decode('utf-8', errors='ignore')
            if self.check_sql_injection(body) or self.check_xss(body):
                abort(400, "Malicious content detected in request body")

# Flask middleware
waf = WAF()

@app.before_request
def security_check():
    waf.validate_request()
```

### 2. DDoS Protection
```python
import time
from collections import defaultdict, deque
import threading

class DDoSProtection:
    def __init__(self):
        self.request_counts = defaultdict(deque)
        self.blocked_ips = set()
        self.lock = threading.Lock()

    def is_ddos_attack(self, client_ip, threshold=100, window=60):
        current_time = time.time()

        with self.lock:
            # Remove old requests
            while (self.request_counts[client_ip] and
                   current_time - self.request_counts[client_ip][0] > window):
                self.request_counts[client_ip].popleft()

            # Add current request
            self.request_counts[client_ip].append(current_time)

            # Check if threshold exceeded
            return len(self.request_counts[client_ip]) > threshold

    def block_ip(self, client_ip, duration=3600):
        """Block IP for specified duration"""
        self.blocked_ips.add(client_ip)

        # Schedule unblock
        def unblock():
            time.sleep(duration)
            self.blocked_ips.discard(client_ip)

        threading.Thread(target=unblock, daemon=True).start()

    def is_blocked(self, client_ip):
        return client_ip in self.blocked_ips

    def check_request(self, client_ip):
        if self.is_blocked(client_ip):
            return False, "IP blocked due to suspicious activity"

        if self.is_ddos_attack(client_ip):
            self.block_ip(client_ip)
            return False, "DDoS attack detected, IP blocked"

        return True, "Request allowed"
```

## Application Security

### 1. Input Validation and Sanitization
```python
import re
import html
from marshmallow import Schema, fields, validate, ValidationError

class SecurityValidator:
    @staticmethod
    def sanitize_html(input_string):
        """Remove potentially dangerous HTML"""
        # Escape HTML entities
        sanitized = html.escape(input_string)

        # Remove script tags and event handlers
        sanitized = re.sub(r'<script[^>]*>.*?</script>', '', sanitized, flags=re.IGNORECASE | re.DOTALL)
        sanitized = re.sub(r'on\w+\s*=\s*["\'][^"\']*["\']', '', sanitized, flags=re.IGNORECASE)

        return sanitized

    @staticmethod
    def validate_sql_safe(input_string):
        """Check for SQL injection patterns"""
        dangerous_patterns = [
            r"(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|EXECUTE)\b)",
            r"(\b(UNION|OR|AND)\b.*\b(SELECT|INSERT|UPDATE|DELETE)\b)",
            r"(--|#|/\*|\*/)",
            r"(\b(xp_|sp_)\w+)",
        ]

        for pattern in dangerous_patterns:
            if re.search(pattern, input_string, re.IGNORECASE):
                raise ValidationError("Input contains potentially dangerous SQL patterns")

        return input_string

class UserRegistrationSchema(Schema):
    username = fields.Str(
        required=True,
        validate=[
            validate.Length(min=3, max=50),
            validate.Regexp(r'^[a-zA-Z0-9_]+$', error="Username can only contain letters, numbers, and underscores")
        ]
    )

    email = fields.Email(required=True)

    password = fields.Str(
        required=True,
        validate=[
            validate.Length(min=8),
            validate.Regexp(
                r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]',
                error="Password must contain uppercase, lowercase, digit, and special character"
            )
        ]
    )

    bio = fields.Str(
        missing="",
        validate=validate.Length(max=500),
        # Custom sanitization
        deserialize=lambda x: SecurityValidator.sanitize_html(x) if x else ""
    )

# Usage
@app.route('/register', methods=['POST'])
def register():
    schema = UserRegistrationSchema()
    try:
        user_data = schema.load(request.json)
        # Process validated and sanitized data
        user = create_user(user_data)
        return jsonify({'message': 'User created successfully'}), 201
    except ValidationError as e:
        return jsonify({'errors': e.messages}), 400
```

### 2. Secure Session Management
```python
import secrets
import time
from datetime import datetime, timedelta

class SecureSessionManager:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.session_timeout = 3600  # 1 hour
        self.max_sessions_per_user = 5

    def create_session(self, user_id, user_agent, ip_address):
        # Generate cryptographically secure session ID
        session_id = secrets.token_urlsafe(32)

        session_data = {
            'user_id': user_id,
            'created_at': time.time(),
            'last_activity': time.time(),
            'user_agent': user_agent,
            'ip_address': ip_address,
            'csrf_token': secrets.token_urlsafe(32)
        }

        # Store session
        self.redis.setex(
            f"session:{session_id}",
            self.session_timeout,
            json.dumps(session_data)
        )

        # Track user sessions
        self.add_user_session(user_id, session_id)

        return session_id, session_data['csrf_token']

    def validate_session(self, session_id, user_agent, ip_address):
        session_data = self.redis.get(f"session:{session_id}")
        if not session_data:
            return None

        session_data = json.loads(session_data)

        # Validate user agent and IP (optional, based on security requirements)
        if session_data['user_agent'] != user_agent:
            self.invalidate_session(session_id)
            return None

        # Update last activity
        session_data['last_activity'] = time.time()
        self.redis.setex(
            f"session:{session_id}",
            self.session_timeout,
            json.dumps(session_data)
        )

        return session_data

    def invalidate_session(self, session_id):
        session_data = self.redis.get(f"session:{session_id}")
        if session_data:
            session_data = json.loads(session_data)
            self.remove_user_session(session_data['user_id'], session_id)

        self.redis.delete(f"session:{session_id}")

    def add_user_session(self, user_id, session_id):
        user_sessions_key = f"user_sessions:{user_id}"
        self.redis.lpush(user_sessions_key, session_id)

        # Limit number of sessions per user
        self.redis.ltrim(user_sessions_key, 0, self.max_sessions_per_user - 1)

        # Set expiration
        self.redis.expire(user_sessions_key, self.session_timeout)
```

### 3. CSRF Protection
```python
import hmac
import hashlib
import time

class CSRFProtection:
    def __init__(self, secret_key):
        self.secret_key = secret_key

    def generate_csrf_token(self, session_id):
        timestamp = str(int(time.time()))
        message = f"{session_id}:{timestamp}"
        signature = hmac.new(
            self.secret_key.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()

        return f"{timestamp}:{signature}"

    def validate_csrf_token(self, token, session_id, max_age=3600):
        try:
            timestamp, signature = token.split(':', 1)

            # Check token age
            if time.time() - int(timestamp) > max_age:
                return False

            # Verify signature
            message = f"{session_id}:{timestamp}"
            expected_signature = hmac.new(
                self.secret_key.encode(),
                message.encode(),
                hashlib.sha256
            ).hexdigest()

            return hmac.compare_digest(signature, expected_signature)

        except (ValueError, TypeError):
            return False

# Flask middleware
csrf = CSRFProtection(app.config['SECRET_KEY'])

@app.before_request
def csrf_protect():
    if request.method in ['POST', 'PUT', 'DELETE', 'PATCH']:
        token = request.headers.get('X-CSRF-Token') or request.form.get('csrf_token')
        session_id = request.cookies.get('session_id')

        if not token or not csrf.validate_csrf_token(token, session_id):
            abort(403, "CSRF token validation failed")
```

## Best Practices

### Security Development Lifecycle
1. **Threat Modeling**: Identify potential threats early in design
2. **Secure Coding**: Follow secure coding practices
3. **Security Testing**: Regular penetration testing and vulnerability assessments
4. **Security Reviews**: Code reviews with security focus
5. **Incident Response**: Prepared response plans for security incidents

### Common Security Vulnerabilities (OWASP Top 10)
1. **Injection**: SQL, NoSQL, OS, and LDAP injection
2. **Broken Authentication**: Session management flaws
3. **Sensitive Data Exposure**: Inadequate protection of sensitive data
4. **XML External Entities (XXE)**: XML processing vulnerabilities
5. **Broken Access Control**: Authorization bypass
6. **Security Misconfiguration**: Default configurations and missing patches
7. **Cross-Site Scripting (XSS)**: Client-side injection attacks
8. **Insecure Deserialization**: Object injection attacks
9. **Using Components with Known Vulnerabilities**: Outdated dependencies
10. **Insufficient Logging & Monitoring**: Inadequate attack detection

### Security Checklist
- [ ] Implement authentication and authorization
- [ ] Encrypt sensitive data at rest and in transit
- [ ] Validate and sanitize all inputs
- [ ] Use HTTPS everywhere
- [ ] Implement proper session management
- [ ] Add CSRF protection
- [ ] Set up security headers
- [ ] Regular security updates
- [ ] Monitor and log security events
- [ ] Conduct regular security assessments

---

**Next Steps:**
- Learn about [Reliability and Fault Tolerance](../reliability/reliability.md)
- Explore [Monitoring and Observability](../reliability/reliability.md#monitoring-and-alerting)
- Study [Case Studies](../case_studies/case_studies.md) with security considerations
```
