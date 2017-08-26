# Elephant Encryption

Elephant Encryption was originally a part of [https://github.com/tagadvance/Gilligan](Gilligan). Please bear in mind that while the encryption itself is very strong, data is only as secure as the server storing the private key.

## Download / Install
The easiest way to install Elephant Encryption is via Composer:
```bash
composer require "tagadvance/elephant:dev-master"
```
```json
{
    "require": {
        "tagadvance/elephant": "dev-master"
    }
}
```

## Example
```php
<?php 

use tagadvance\elephant\cryptography\CertificateSigningRequest;
use tagadvance\elephant\cryptography\ConfigurationBuilder;
use tagadvance\elephant\cryptography\PrivateKey;
use tagadvance\elephant\cryptography\PublicKey;
use tagadvance\elephant\cryptography\PrivateKeyCryptographer;
use tagadvance\elephant\cryptography\PublicKeyCryptographer;
use tagadvance\elephant\base\Standard;
use tagadvance\elephant\security\Hash;
use tagadvance\elephant\cryptography\DistinguishedNameBuilder;
use tagadvance\elephant\cryptography\Base64Cryptographer;

require_once 'vendor/autoload.php';

$data = <<<DATA
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad 
minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit 
in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia 
deserunt mollit anim id est laborum.
DATA;

$configuration = ConfigurationBuilder::builder()
		->setConfigurationFile(ConfigurationBuilder::CONFIG_DEBIAN)
		->setDigestAlgorithm(Hash::ALGORITHM_SHA512)
		->setX509Extensions('v3_ca')
		->setRequiredExtensions('v3_req')
		->setPrivateKeyBits(4096)
		->setPrivateKeyType();
$privateKey = PrivateKey::newPrivateKey($configuration);
try {
	$dn = DistinguishedNameBuilder::builder()
			->setCountryName('US')
			->setStateOrProvinceName('OR')
			->setLocality('Crater Lake')
			->setOrganizationName('Acme Corporation')
			->setOrganizationUnitName ( '.' )
			->setCommonName ( 'Wile E Coyote' )
			->setEmailAddress ( 'w.coyote@acme.com');
	$csr = CertificateSigningRequest::newCertificateSigningRequest($dn, $privateKey);
	$certificate = $csr->sign($privateKey);
	try {
		$publicKey = PublicKey::createFromCertificate($certificate);
		
		$privateKeyCryptographer = Base64Cryptographer::create ( new PrivateKeyCryptographer ( $privateKey ) );
		$encryptedData = $privateKeyCryptographer->encrypt($data);
		Standard::output()->printLine($encryptedData);
		
		$publicKeyCryptographer = Base64Cryptographer::create(new PublicKeyCryptographer( $privateKey, $publicKey ));
		
		$decryptedData = $publicKeyCryptographer->decrypt($encryptedData);
		Standard::output()->printLine($decryptedData);
		
		$encryptedData = $publicKeyCryptographer->encrypt($data);
		Standard::output()->printLine($encryptedData);
		
		$decryptedData = $privateKeyCryptographer->decrypt($encryptedData);
		Standard::output()->printLine($decryptedData);
	} finally {
		$certificate->close();
	}
} finally {
	$privateKey->close();
}
```