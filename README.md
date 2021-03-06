- š Hi, Iām @genigre
- š Iām interested in ...
- š± Iām currently learning ...
- šļø Iām looking to collaborate on ...
- š« How to reach me ...

<!---
genigre/genigre is a āØ special āØ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
"""
Base45 Data Encoding as described in draft-faltstrom-base45-02
"""

from typing import Union

BASE45_CHARSET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ $%*+-./:"
BASE45_DICT = {v: i for i, v in enumerate(BASE45_CHARSET)}


def b45encode(buf: bytes) -> bytes:
    """Convert bytes to base45-encoded string"""
    res = ""
    buflen = len(buf)
    for i in range(0, buflen & ~1, 2):
        x = (buf[i] << 8) + buf[i + 1]
        e, x = divmod(x, 45 * 45)
        d, c = divmod(x, 45)
        res += BASE45_CHARSET[c] + BASE45_CHARSET[d] + BASE45_CHARSET[e]
    if buflen & 1:
        d, c = divmod(buf[-1], 45)
        res += BASE45_CHARSET[c] + BASE45_CHARSET[d]
    return res.encode()


def b45decode(s: Union[bytes, str]) -> bytes:
    """Decode base45-encoded string to bytes"""
    try:
        if isinstance(s, str):
            buf = [BASE45_DICT[c] for c in s.rstrip("\n")]
        elif isinstance(s, bytes):
            buf = [BASE45_DICT[c] for c in s.decode()]
        else:
            raise TypeError("Type must be 'str' or 'bytes'")

        buflen = len(buf)
        if buflen % 3 == 1:
            raise ValueError("Invalid base45 string")

        res = []
        for i in range(0, buflen, 3):
            if buflen - i >= 3:
                x = buf[i] + buf[i + 1] * 45 + buf[i + 2] * 45 * 45
                if x > 0xFFFF:
                    raise ValueError
                res.extend(divmod(x, 256))
            else:
                x = buf[i] + buf[i + 1] * 45
                if x > 0xFF:
                    raise ValueError
                res.append(x)
        return bytes(res)
    except (ValueError, KeyError, AttributeError):
        raise ValueError("Invalid base45 string")
