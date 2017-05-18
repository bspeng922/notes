# Django 默认密码加密算法

## 生成密码

Creates a hashed password in the format used by this application. It takes one mandatory argument: the password in plain-text. Optionally, you can provide a salt and a hashing algorithm to use.

```
# 设置密码
def set_password(self, raw_password):
    self.password = make_password(raw_password)
    self._password = raw_password

# 生成密码
def make_password(password, salt=None, hasher='default'):
    """
    Turn a plain-text password into a hash for database storage

    Same as encode() but generates a new random salt.
    If password is None then a concatenation of
    UNUSABLE_PASSWORD_PREFIX and a random string will be returned
    which disallows logins. Additional random string reduces chances
    of gaining access to staff or superuser accounts.
    See ticket #20079 for more info.
    """
    if password is None:
        return UNUSABLE_PASSWORD_PREFIX + get_random_string(UNUSABLE_PASSWORD_SUFFIX_LENGTH)
    hasher = get_hasher(hasher)

    if not salt:
        salt = hasher.salt()

    return hasher.encode(password, salt)
```

如果salt为None，则每次生成的密码会不一样，如果你不想每次都生成不同的密文，可以把make_password的salt字段给一个固定的字符串，比如：

```
>>> make_password(text, "a", 'pbkdf2_sha256')
u'pbkdf2_sha256$12000$a$5HkIPczRZGSTKUBa5uzZmRuAWdp2Qe6Oemhdasvzv4Q='
>>> make_password(text, "a", 'pbkdf2_sha256')
u'pbkdf2_sha256$12000$a$5HkIPczRZGSTKUBa5uzZmRuAWdp2Qe6Oemhdasvzv4Q='
```

make_password第三个参数hasher是表示生成密文的一种方式，根据文档给出的大概有这几种：
pbkdf2_sha256
pbkdf2_sha1
bcrypt_sha256
bcrypt
sha1
unsalted_md5
crypt

就是调用hasher的encode(password, salt)方法来生成密码。


## 检查密码

If you’d like to manually authenticate a user by comparing a plain-text password to the hashed password in the database, use the convenience function check_password().
It takes two arguments: the plain-text password to check, and the full value of a user’s password field in the database to check against, and returns True if they match, False otherwise.

```
# 检查密码是否合法
def check_password(self, raw_password):
    """
    Return a boolean of whether the raw_password was correct. Handles
    hashing formats behind the scenes.
    """
    def setter(raw_password):
        self.set_password(raw_password)
        # Password hash upgrades shouldn't be considered password changes.
        self._password = None
        self.save(update_fields=["password"])
    return check_password(raw_password, self.password, setter)

# 检查密码
def check_password(password, encoded, setter=None, preferred='default'):
    """
    Returns a boolean of whether the raw password matches the three
    part encoded digest.

    If setter is specified, it'll be called when you need to
    regenerate the password.
    """
    if password is None or not is_password_usable(encoded):
        return False

    preferred = get_hasher(preferred)
    hasher = identify_hasher(encoded)

    hasher_changed = hasher.algorithm != preferred.algorithm
    must_update = hasher_changed or preferred.must_update(encoded)
    is_correct = hasher.verify(password, encoded)

    # If the hasher didn't change (we don't protect against enumeration if it
    # does) and the password should get updated, try to close the timing gap
    # between the work factor of the current encoded password and the default
    # work factor.
    if not is_correct and not hasher_changed and must_update:
        hasher.harden_runtime(password, encoded)

    if setter and is_correct and must_update:
        setter(password)
    return is_correct

```







