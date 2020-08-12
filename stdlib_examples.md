
# Method

The rationale for PEP 622 states that "Much of the code to do so tends to consist of complex chains of nested if/elif statements, including multiple calls to len(), isinstance() and index/key/attribute access."

I ran two different queries over the standard library, `Tools` and `doc` folders, over 630 000 lines of code in total.

The query finds all if statements with at least two tests of the same variable, where the tests involve `isinstance` or `len`, and where some destrucuring of that variable occurs in the body of the `if` statement.
Destructuring of the variable `var` means any of the following:

* `x, ... = var`
* `y = x[...]`
* `z = var.attr`

https://lgtm.com/query/7291159440077780290/

There are 24 results, of which a few are false positives, leaving the following that could potentially have the test and destructuring combined:

https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Tools/pynche/Main.py?sort=name&dir=ASC&mode=heatmap#L192
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/argparse.py?sort=name&dir=ASC&mode=heatmap#L2206
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/ast.py?sort=name&dir=ASC&mode=heatmap#L284
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Tools/peg_generator/pegen/build.py?sort=name&dir=ASC&mode=heatmap#L131


https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/xmlrpc/client.py?sort=name&dir=ASC&mode=heatmap#L303

https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/configparser.py?sort=name&dir=ASC&mode=heatmap#L495
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/dataclasses.py?sort=name&dir=ASC&mode=heatmap#L1207
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Tools/scripts/db2pickle.py?sort=name&dir=ASC&mode=heatmap#L59
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/xml/dom/expatbuilder.py?sort=name&dir=ASC&mode=heatmap#L118
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/distutils/fancy_getopt.py?sort=name&dir=ASC&mode=heatmap#L144

https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/fractions.py?sort=name&dir=ASC&mode=heatmap#L96
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/imaplib.py?sort=name&dir=ASC&mode=heatmap#L1516

https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/unittest/mock.py?sort=name&dir=ASC&mode=heatmap#L90
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Doc/tools/rstlint.py?sort=name&dir=ASC&mode=heatmap#L160
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/smtpd.py?sort=name&dir=ASC&mode=heatmap#L906
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/test/support/socket_helper.py?sort=name&dir=ASC&mode=heatmap#L255

https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/turtle.py?sort=name&dir=ASC&mode=heatmap#L1852
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/turtle.py?sort=name&dir=ASC&mode=heatmap#L1884
https://lgtm.com/projects/g/python/cpython/snapshot/d39328cb48ce1e51ecd377c8b727c91cc8735eb0/files/Lib/typing.py?sort=name&dir=ASC&mode=heatmap#L710

For each example I have shown the (trimmed) original, and the same code rewritten using PEP 622 matching, and (where relevant) using `type.__contains__`, or a simple `switch` statement, along the lines of PEP 275, for comparison.

I have removed comments from the original to avoid obsuring the syntax.

# Examples

## Tools/pynch/Main.py main()

### Original

```python
    if len(args) == 0:
        initialcolor = None
    elif len(args) == 1:
        initialcolor = args[0]
    else:
        usage(1)
```

### PEP 622

```python
    match args:
        case []:
            initialcolor = None
        case [initialcolor]:
            pass
        case _:
            usage(1)
```

### Switch

```python
    switch len(args):
        case 0:
          initialcolor = None
        case 1:
            initialcolor = args[0]
        else:
            usage(1)  
```

## argparse.ArgumentParser._parse_optional

### Original

```python
    if len(option_tuples) > 1:
        options = ', '.join([option_string
            for action, option_string, explicit_arg in option_tuples])
        args = {'option': arg_string, 'matches': options}
        msg = _('ambiguous option: %(option)s could match %(matches)s')
        self.error(msg % args)
    elif len(option_tuples) == 1:
        option_tuple, = option_tuples
        return option_tuple
```

### PEP 622

```python
    match option_tuples:
        case []:
            pass
        case [option_tuple]:
            return option_tuple
        case _:
            options = ', '.join([option_string
                for action, option_string, explicit_arg in option_tuples])
            args = {'option': arg_string, 'matches': options}
            msg = _('ambiguous option: %(option)s could match %(matches)s')
            self.error(msg % args)
```

### Switch

```python
    switch len(option_tuples):
        case 0:
            pass
        case 1:
            option_tuple, = option_tuples
            return option_tuples[0]
        else:
            options = ', '.join([option_string
                for action, option_string, explicit_arg in option_tuples])
            args = {'option': arg_string, 'matches': options}
            msg = _('ambiguous option: %(option)s could match %(matches)s')
            self.error(msg % args)
```


## ast.get_docstring

### Original

```python
    if isinstance(node, Str):
        text = node.s
    elif isinstance(node, Constant) and isinstance(node.value, str):
        text = node.value
    else:
        return None
```

### PEP 622

```python
    match node:
        case Str(s):
            text = s
        case Node(value=str()):
            text = value
        case _:
            return None
```

### type.__contains__

```python
    if node in Str:
        text = node.s
    elif node in Constant and node.value in str:
        text = node.value
    else:
        return None

```

## Tools/peg_generator/pegen/build.py generate_token_definitions()

### Original

```python
    if len(pieces) == 1:
        (token,) = pieces
        non_exact_tokens.add(token)
        all_tokens[index] = token
    elif len(pieces) == 2:
        token, op = pieces
        exact_tokens[op.strip("'")] = index
        all_tokens[index] = token
    else:
        raise ValueError(f"Unexpected line found in Tokens file: {line}")
```

### PEP 622

```python
    match pieces:
        case [token]:
            non_exact_tokens.add(token)
            all_tokens[index] = token
        case [token, op]:
            exact_tokens[op.strip("'")] = index
            all_tokens[index] = token
        case _:
            raise ValueError(f"Unexpected line found in Tokens file: {line}")
```

### Alternative implementation
```python
    if len(pieces) in (1,2):
        token, *op = pieces:
        if op:
            exact_tokens[op[0].strip("'")] = index
        else:
            non_exact_tokens.add(token)
        all_tokens[index] = token
    else:
        raise ValueError(f"Unexpected line found in Tokens file: {line}")
```

### Switch

```python
    switch len(pieces):
        case 1:
            (token,) = pieces
            non_exact_tokens.add(token)
            all_tokens[index] = token
        case 2:
            token, op = pieces
            exact_tokens[op.strip("'")] = index
            all_tokens[index] = token
        else:
            raise ValueError(f"Unexpected line found in Tokens file: {line}")

```


## xmlrpc.client.DateTime.make_comparable

### Original

```python
    if isinstance(other, DateTime):
        s = self.value
        o = other.value
    elif isinstance(other, datetime):
        s = self.value
        o = _iso8601_format(other)
    elif isinstance(other, str):
        s = self.value
        o = other
    elif hasattr(other, "timetuple"):
        s = self.timetuple()
        o = other.timetuple()
    else:
        s = self
        o = NotImplemented
```

### PEP 622

```python
    match other:
        case DateTime(value):
            s = self.value
            o = value
        case datetime():
            s = self.value
            o = _iso8601_format(other)
        case str():
            s = self.value
            o = other
        case _:
            if hasattr(other, "timetuple"):
                s = self.timetuple()
                o = other.timetuple()
            else:
                s = self
                o = NotImplemented
```

### type.__contains__

```python
    if other in DateTime:
        s = self.value
        o = other.value
    elif other in datetime:
        s = self.value
        o = _iso8601_format(other)
    elif other in str:
        s = self.value
        o = other
    elif hasattr(other, "timetuple"):
        s = self.timetuple()
        o = other.timetuple()
    else:
        s = self
        o = NotImplemented
```

## configparder.ExtendedInterpolation._interpolate_some

### Original

```python
    if len(path) == 1:
        opt = parser.optionxform(path[0])
        v = map[opt]
    elif len(path) == 2:
        sect = path[0]
        opt = parser.optionxform(path[1])
        v = parser.get(sect, opt, raw=True)
    else:
        raise InterpolationSyntaxError(
            option, section,
            "More than one ':' found: %r" % (rest,))
```

### PEP 622

```python
    match path:
        case [optionstr]:
            opt = parser.optionxform(optionstr)
            v = map[opt]
        case [sect, optionstr]:
            opt = parser.optionxform(optionstr)
            v = parser.get(sect, opt, raw=True)
        case _:
            raise InterpolationSyntaxError(
                option, section,
                "More than one ':' found: %r" % (rest,))

```

### Switch

```python
    switch len(path):
        case 1:
            opt = parser.optionxform(path[0])
            v = map[opt]
        case 2:
            sect = path[0]
            opt = parser.optionxform(path[1])
            v = parser.get(sect, opt, raw=True)
        else:
            raise InterpolationSyntaxError(
                option, section,
                "More than one ':' found: %r" % (rest,))
```

## dataclasses.make_dataclass

### Original

```python
    if isinstance(item, str):
        name = item
        tp = 'typing.Any'
    elif len(item) == 2:
        name, tp, = item
    elif len(item) == 3:
        name, tp, spec = item
        namespace[name] = spec
    else:
        raise TypeError(f'Invalid field: {item!r}')
```

### PEP 622

```python
    match item:
        case str():
            name = item
            tp = 'typing.Any'
        case [name, tp]:
            pass
        case [name, tp, spec]:
            namespace[name] = spec
        case _:
            raise TypeError(f'Invalid field: {item!r}')
  
```

### type.__contains__

```python
    if item in str:
        name = item
        tp = 'typing.Any'
    elif len(item) == 2:
        name, tp, = item
    elif len(item) == 3:
        name, tp, spec = item
        namespace[name] = spec
    else:
        raise TypeError(f'Invalid field: {item!r}')
```


## Tools/scripts/db2pickle.py main()

### Original

```python
    if len(args) == 0 or len(args) > 2:
        usage()
        return 1
    elif len(args) == 1:
        dbfile = args[0]
        pfile = sys.stdout
    else:
        dbfile = args[0]
        try:
            pfile = open(args[1], 'wb')
        except IOError:
            sys.stderr.write("Unable to open %s\n" % args[1])
            return 1
```

### PEP 622

```python
    match args:
        case [dbfile]:
            pfile = sys.stdout
        case [dbfile, filename]:
            try:
                pfile = open(filename), 'wb')
            except IOError:
                sys.stderr.write("Unable to open %s\n" % filename)
                return 1
        case _:
            usage()
            return 1
```

### Switch

```python
    switch len(args):
        case 1:
            dbfile = args[0]
            pfile = sys.stdout
        case 2:
            dbfile, filename = args
            try:
                pfile = open(filename), 'wb')
            except IOError:
                sys.stderr.write("Unable to open %s\n" % filename)
                return 1
        else:
            usage()
            return 1

```

## xml.dom.expatbuilder._parse_ns_name

### Original

```python
    if len(parts) == 3:
        uri, localname, prefix = parts
        prefix = intern(prefix, prefix)
        qname = "%s:%s" % (prefix, localname)
        qname = intern(qname, qname)
        localname = intern(localname, localname)
    elif len(parts) == 2:
        uri, localname = parts
        prefix = EMPTY_PREFIX
        qname = localname = intern(localname, localname)
    else:
        raise ValueError("Unsupported syntax: spaces in URIs not supported: %r" % name)
```

### PEP 622

```python
    match parts:
        case [uri, localname, prefix]:
            prefix = intern(prefix, prefix)
            qname = "%s:%s" % (prefix, localname)
            qname = intern(qname, qname)
            localname = intern(localname, localname)
        case [uri, localname]:
            prefix = EMPTY_PREFIX
            qname = localname = intern(localname, localname)
        case _:
            raise ValueError("Unsupported syntax: spaces in URIs not supported: %r" % name)
```

## distutils.fancy_getopt.FancyGetopt._grok_option_table

### Original

```python
    if len(option) == 3:
        long, short, help = option
        repeat = 0
    elif len(option) == 4:
        long, short, help, repeat = option
    else:
        # the option table is part of the code, so simply
        # assert that it is correct
        raise ValueError("invalid option tuple: %r" % (option,))
```

### PEP 622

```python
    match option:
        case [long, short, help]:
            repeat = 0
        case [long, short, help, repeat]:
            pass
        case _:
            # the option table is part of the code, so simply
            # assert that it is correct
            raise ValueError("invalid option tuple: %r" % (option,))
```

### Alternative Python implementation

```python
    if len(option) in (3, 4):
        long, short, help, *opt_repeat = option
        repeat = opt_repeat[0] if opt_repeat else 0
    else:
        raise ValueError("invalid option tuple: %r" % (option,))
```


## fractions.Fraction.__new__

### Original

```python
    if type(numerator) is int:
        self._numerator = numerator
        self._denominator = 1
        return self
    elif isinstance(numerator, numbers.Rational):
        self._numerator = numerator.numerator
        self._denominator = numerator.denominator
        return self
    elif isinstance(numerator, (float, Decimal)):
        # Exact conversion
        self._numerator, self._denominator = numerator.as_integer_ratio()
        return self
    elif isinstance(numerator, str):
        # Handle construction from strings.
        ... # Trimmed, as no destructuring occurs
    else:
        raise TypeError("argument should be a string "
                        "or a Rational instance")
```

### PEP 622

```python
    match numerator:
        case _ if type(numerator) == int:
            self._numerator = numerator
            self._denominator = 1
            return self
        case numbers.Rational(numerator, demoninator)):
            self._numerator = numerator
            self._denominator = denominator
            return self
        case float() | Decimal():
            # Exact conversion
            self._numerator, self._denominator = numerator.as_integer_ratio()
            return self
        case str():
            # Handle construction from strings.
            ... # Trimmed, as no destructuring occurs
        case _:
            raise TypeError("argument should be a string "
                            "or a Rational instance")
```

Note that the numerator must be an `int`, not a subclass, so a class pattern doesn't work.
Also, not the potential confusion as the variable and `Rational` attribute name are the same.

### type.__contains__

```python
    if type(numerator) is int:
        self._numerator = numerator
        self._denominator = 1
        return self
    elif numerator in numbers.Rational:
        self._numerator = numerator.numerator
        self._denominator = numerator.denominator
        return self
    elif in float or numerator in Decimal:
        # Exact conversion
        self._numerator, self._denominator = numerator.as_integer_ratio()
        return self
    elif numerator in str:
        # Handle construction from strings.
        ... # Trimmed, as no destructuring occurs
    else:
        raise TypeError("argument should be a string "
                        "or a Rational instance")
```


## imaplib.Time2Internaldate

### Original

```python
    if isinstance(date_time, (int, float)):
        dt = datetime.fromtimestamp(date_time,
                                    timezone.utc).astimezone()
    elif isinstance(date_time, tuple):
        try:
            gmtoff = date_time.tm_gmtoff
        except AttributeError:
            if time.daylight:
                dst = date_time[8]
                if dst == -1:
                    dst = time.localtime(time.mktime(date_time))[8]
                gmtoff = -(time.timezone, time.altzone)[dst]
            else:
                gmtoff = -time.timezone
        delta = timedelta(seconds=gmtoff)
        dt = datetime(*date_time[:6], tzinfo=timezone(delta))
    elif isinstance(date_time, datetime):
        if date_time.tzinfo is None:
            raise ValueError("date_time must be aware")
        dt = date_time
    elif isinstance(date_time, str) and (date_time[0],date_time[-1]) == ('"','"'):
        return date_time        # Assume in correct format
    else:
        raise ValueError("date_time not of a known type")
```

### PEP 622

```python
    match date_time:
        case int() | float():
            dt = datetime.fromtimestamp(date_time,
                                        timezone.utc).astimezone()
        case tuple(tm_gmtoff):
            gmtoff = tm_gmtoff
            delta = timedelta(seconds=gmtoff)
            dt = datetime(*date_time[:6], tzinfo=timezone(delta))
        case tuple():
            if time.daylight:
                dst = date_time[8]
                if dst == -1:
                    dst = time.localtime(time.mktime(date_time))[8]
                gmtoff = -(time.timezone, time.altzone)[dst]
            else:
                gmtoff = -time.timezone
            delta = timedelta(seconds=gmtoff)
            dt = datetime(*date_time[:6], tzinfo=timezone(delta))
        case datetime():
            if date_time.tzinfo is None:
                raise ValueError("date_time must be aware")
            dt = date_time
        case str() if (date_time[0],date_time[-1]) == ('"','"'):
            return date_time        # Assume in correct format
        case _:
            raise ValueError("date_time not of a known type")
```

### type.__contains__

```python
    if date_time in int or date_time in float:
        dt = datetime.fromtimestamp(date_time,
                                    timezone.utc).astimezone()
    elif date_time in tuple:
        try:
            gmtoff = date_time.tm_gmtoff
        except AttributeError:
            if time.daylight:
                dst = date_time[8]
                if dst == -1:
                    dst = time.localtime(time.mktime(date_time))[8]
                gmtoff = -(time.timezone, time.altzone)[dst]
            else:
                gmtoff = -time.timezone
        delta = timedelta(seconds=gmtoff)
        dt = datetime(*date_time[:6], tzinfo=timezone(delta))
    elif date_time in  datetime:
        if date_time.tzinfo is None:
            raise ValueError("date_time must be aware")
        dt = date_time
    elif date_time in str and (date_time[0],date_time[-1]) == ('"','"'):
        return date_time        # Assume in correct format
    else:
        raise ValueError("date_time not of a known type")
```


## unittest.mock._get_signature_object

### Original

```python
    if isinstance(func, type) and not as_instance:
        # If it's a type and should be modelled as a type, use __init__.
        func = func.__init__
        # Skip the `self` argument in __init__
        eat_self = True
    elif not isinstance(func, FunctionTypes):
        # If we really want to model an instance of the passed type,
        # __call__ should be looked up, not __init__.
        try:
            func = func.__call__
        except AttributeError:
            return None
```

### PEP 622

```python
    match func:
        case type() if not as_instance:
            # If it's a type and should be modelled as a type, use __init__.
            func = func.__init__
            # Skip the `self` argument in __init__
            eat_self = True
        case _ if not isinstance(func, FunctionTypes):
            # If we really want to model an instance of the passed type,
            # __call__ should be looked up, not __init__.
            try:
                func = func.__call__
            except AttributeError:
                return None
```

### type.__contains__

```python
   if func in type and not as_instance:
        # If it's a type and should be modelled as a type, use __init__.
        func = func.__init__
        # Skip the `self` argument in __init__
        eat_self = True
    elif func not in FunctionTypes:
        # If we really want to model an instance of the passed type,
        # __call__ should be looked up, not __init__.
        try:
            func = func.__call__
        except AttributeError:
            return None
```


## Doc/tools/rstlint.py main()

### Original

```python
    if len(args) == 0:
        path = '.'
    elif len(args) == 1:
        path = args[0]
    else:
        print(usage)
        return 2
```

### PEP 622

```python
    match args:
        case []:
            path = '.'
        case [path]:
            pass
        case _:
            print(usage)
            return 2
```

### Switch

```python
    switch len(args):
        case 0:
            path = '.'
        case 1:
            path = args[0]
        else:
            print(usage)
            return 2
```

## smtpd.parseargs

### Original

```python
    if len(args) < 1:
        localspec = 'localhost:8025'
        remotespec = 'localhost:25'
    elif len(args) < 2:
        localspec = args[0]
        remotespec = 'localhost:25'
    elif len(args) < 3:
        localspec = args[0]
        remotespec = args[1]
    else:
        usage(1, 'Invalid arguments: %s' % COMMASPACE.join(args))
 
```

### PEP 622

```python
    match args:
        case []:
            localspec = 'localhost:8025'
            remotespec = 'localhost:25'
        case [localspec]:
            remotespec = 'localhost:25'
        case [localspec, remotespec]:
            pass
        else:
            usage(1, 'Invalid arguments: %s' % COMMASPACE.join(args))
```

### Alternative Python implementation

```python
    remotespec = 'localhost:25'
    if len(args) < 1:
        localspec = 'localhost:8025'
    elif len(args) < 2:
        localspec = args[0]
    elif len(args) < 3:
        localspec = args[0]
        remotespec = args[1]
    else:
        usage(1, 'Invalid arguments: %s' % COMMASPACE.join(args))
```

## test.support.socket_helper.transient_internet

### Original

```python
    if len(a) >= 1 and isinstance(a[0], OSError):
        err = a[0]
    elif len(a) >= 2 and isinstance(a[1], OSError):
        err = a[1]
    else:
        break
```

### PEP 622

```python
    match a:
        case [err:=OSError(), *_]:
            pass
        case [_, err:=OSError(), _, *_]:
            err = a[1]
        case _:
            break

```

### type.__contains__

```python
    if len(a) >= 1 and a[0] in OSError:
        err = a[0]
    elif len(a) >= 2 and a[1] in OSError:
        err = a[1]
    else:
        break

```

## turtle.TNavigator.distance and turtle.TNavigator.towards

Both `turtle.TNavigator.distance` and `turtle.TNavigator.towards` contain the same if statement.

### Original

```python
    if isinstance(x, Vec2D):
        pos = x
    elif isinstance(x, tuple):
        pos = Vec2D(*x)
    elif isinstance(x, TNavigator):
        pos = x._position
```

### PEP 622

```python
    match x:
        case Vec2D():
            pos = x
        case tuple():
            pos = Vec2D(*x)
        case TNavigator(_position):
            pos = _position
```

### type.__contains__

```python
    if x in Vec2D:
        pos = x
    elif x in tuple:
        pos = Vec2D(*x)
    elif x in TNavigator:
        pos = x._position
```

## typing.GenericAlias.__getitem__

### Original

```python
    if isinstance(arg, TypeVar):
        arg = subst[arg]
    elif isinstance(arg, (_GenericAlias, GenericAlias)):
        subparams = arg.__parameters__
        if subparams:
            subargs = tuple(subst[x] for x in subparams)
            arg = arg[subargs]
```

### PEP 622

```python
    match arg:
        case TypeVar():
            arg = subst[arg]
        case _GenericAlias(__parameters__) | GenericAlias(__parameters__):
            subparams = __parameters__
            if subparams:
                subargs = tuple(subst[x] for x in subparams)
                arg = arg[subargs]
```

### type.__contains__

```python
    if arg in TypeVar:
        arg = subst[arg]
    elif arg in _GenericAlias or arg in GenericAlias:
        subparams = arg.__parameters__
        if subparams:
            subargs = tuple(subst[x] for x in subparams)
            arg = arg[subargs]
```


# Conclusions

The 18 examples can be broken down into the following categories:

* Tests on length only: 10 cases
* Test on type only: 5 cases
* Tests on both type and length: 1 case
* Test on either type or length and some other condition: 2 cases

The case where testing is done on both type and length (`dataclasses.make_dataclass`) is
the only case where PEP 622 shows an improvement in readability.
It would appear that the benefits of PEP 622 are slight.

That over half the cases test for a narrow range of sequence lengths suggests that a zero-or-one
unpacking syntax might be useful; `?name` as an alternative to `*name`.
`?name` would match zero or one items, as opposed to `*name` which matches zero or more.

Using this extension, the `smtpd.parseargs` example could be rewritten as:

```python
    localspec = 'localhost:8025'
    remotespec = 'localhost:25'
    try:
        ?localspec, ?remotespec = args
    except ValueError:
        usage(1, 'Invalid arguments: %s' % COMMASPACE.join(args))
```


