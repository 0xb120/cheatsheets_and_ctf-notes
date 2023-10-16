---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - code-review
  - deobfuscation
  - reversing
---
>[!quote]
>*We want to update our website but we are unable to because the developer who coded this left today. Can you take a look?* 

# Set up

-

# Information Gathering

Contents of the folder:

![Pasted image 20210528122004.png](../../zzz_res/attachments/Pasted_image_20210528122004.png)

**login.js:**

```jsx
var _0x4e0b=['\x74\x6f\x53\x74\x72\x69\x6e\x67','\x75\x73\x65\x72\x6e\x61\x6d\x65','\x63\x6f\x6e\x73\x6f\x6c\x65','\x67\x65\x74\x45\x6c\x65\x6d\x65\x6e\x74\x42\x79\x49\x64','\x6c\x6f\x67','\x62\x69\x6e\x64','\x64\x69\x73\x61\x62\x6c\x65\x64','\x61\x70\x70\x6c\x79','\x61\x64\x6d\x69\x6e','\x70\x72\x6f\x74\x6f\x74\x79\x70\x65','\x7b\x7d\x2e\x63\x6f\x6e\x73\x74\x72\x75\x63\x74\x6f\x72\x28\x22\x72\x65\x74\x75\x72\x6e\x20\x74\x68\x69\x73\x22\x29\x28\x20\x29','\x20\x61\x74\x74\x65\x6d\x70\x74\x3b','\x76\x61\x6c\x75\x65','\x63\x6f\x6e\x73\x74\x72\x75\x63\x74\x6f\x72','\x59\x6f\x75\x20\x68\x61\x76\x65\x20\x6c\x65\x66\x74\x20','\x74\x72\x61\x63\x65','\x72\x65\x74\x75\x72\x6e\x20\x2f\x22\x20\x2b\x20\x74\x68\x69\x73\x20\x2b\x20\x22\x2f','\x74\x61\x62\x6c\x65','\x6c\x65\x6e\x67\x74\x68','\x5f\x5f\x70\x72\x6f\x74\x6f\x5f\x5f','\x65\x72\x72\x6f\x72','\x4c\x6f\x67\x69\x6e\x20\x73\x75\x63\x63\x65\x73\x73\x66\x75\x6c\x6c\x79'];(function(_0x173c04,_0x4e0b6e){var _0x20fedb=function(_0x2548ec){while(--_0x2548ec){_0x173c04['\x70\x75\x73\x68'](_0x173c04['\x73\x68\x69\x66\x74']());}},_0x544f36=function(){var _0x4c641a={'\x64\x61\x74\x61':{'\x6b\x65\x79':'\x63\x6f\x6f\x6b\x69\x65','\x76\x61\x6c\x75\x65':'\x74\x69\x6d\x65\x6f\x75\x74'},'\x73\x65\x74\x43\x6f\x6f\x6b\x69\x65':function(_0x35c856,_0x13e7c5,_0x58186,_0xf5e7a4){_0xf5e7a4=_0xf5e7a4||{};var _0x120843=_0x13e7c5+'\x3d'+_0x58186,_0x3f3096=0x0;for(var _0x159a78=0x0,_0x1307a5=_0x35c856['\x6c\x65\x6e\x67\x74\x68'];_0x159a78<_0x1307a5;_0x159a78++){var _0x2316f9=_0x35c856[_0x159a78];_0x120843+='\x3b\x20'+_0x2316f9;var _0x22cb86=_0x35c856[_0x2316f9];_0x35c856['\x70\x75\x73\x68'](_0x22cb86),_0x1307a5=_0x35c856['\x6c\x65\x6e\x67\x74\x68'],_0x22cb86!==!![]&&(_0x120843+='\x3d'+_0x22cb86);}_0xf5e7a4['\x63\x6f\x6f\x6b\x69\x65']=_0x120843;},'\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65':function(){return'\x64\x65\x76';},'\x67\x65\x74\x43\x6f\x6f\x6b\x69\x65':function(_0x589958,_0x2bfede){_0x589958=_0x589958||function(_0x168695){return _0x168695;};var _0x4b3aae=_0x589958(new RegExp('\x28\x3f\x3a\x5e\x7c\x3b\x20\x29'+_0x2bfede['\x72\x65\x70\x6c\x61\x63\x65'](/([.$?*|{}()[]\/+^])/g,'\x24\x31')+'\x3d\x28\x5b\x5e\x3b\x5d\x2a\x29')),_0x43e750=function(_0x387366,_0x8c72e7){_0x387366(++_0x8c72e7);};return _0x43e750(_0x20fedb,_0x4e0b6e),_0x4b3aae?decodeURIComponent(_0x4b3aae[0x1]):undefined;}},_0x1d30b3=function(){var _0x23ed4e=new RegExp('\x5c\x77\x2b\x20\x2a\x5c\x28\x5c\x29\x20\x2a\x7b\x5c\x77\x2b\x20\x2a\x5b\x27\x7c\x22\x5d\x2e\x2b\x5b\x27\x7c\x22\x5d\x3b\x3f\x20\x2a\x7d');return _0x23ed4e['\x74\x65\x73\x74'](_0x4c641a['\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65']['\x74\x6f\x53\x74\x72\x69\x6e\x67']());};_0x4c641a['\x75\x70\x64\x61\x74\x65\x43\x6f\x6f\x6b\x69\x65']=_0x1d30b3;var _0x488f18='';var _0x4ac08e=_0x4c641a['\x75\x70\x64\x61\x74\x65\x43\x6f\x6f\x6b\x69\x65']();if(!_0x4ac08e)_0x4c641a['\x73\x65\x74\x43\x6f\x6f\x6b\x69\x65'](['\x2a'],'\x63\x6f\x75\x6e\x74\x65\x72',0x1);else _0x4ac08e?_0x488f18=_0x4c641a['\x67\x65\x74\x43\x6f\x6f\x6b\x69\x65'](null,'\x63\x6f\x75\x6e\x74\x65\x72'):_0x4c641a['\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65']();};_0x544f36();}(_0x4e0b,0x182));var _0x20fe=function(_0x173c04,_0x4e0b6e){_0x173c04=_0x173c04-0x0;var _0x20fedb=_0x4e0b[_0x173c04];return _0x20fedb;};var _0x35c856=function(){var _0x58186=!![];return function(_0xf5e7a4,_0x120843){var _0x3f3096=_0x58186?function(){var _0x228e0e=_0x20fe;if(_0x120843){var _0x159a78=_0x120843[_0x228e0e('\x30\x78\x31\x31')](_0xf5e7a4,arguments);return _0x120843=null,_0x159a78;}}:function(){};return _0x58186=![],_0x3f3096;};}(),_0x4ac08e=_0x35c856(this,function(){var _0x1307a5=function(){var _0x257462=_0x20fe,_0x2316f9=_0x1307a5[_0x257462('\x30\x78\x31')](_0x257462('\x30\x78\x34'))()[_0x257462('\x30\x78\x31')]('\x5e\x28\x5b\x5e\x20\x5d\x2b\x28\x20\x2b\x5b\x5e\x20\x5d\x2b\x29\x2b\x29\x2b\x5b\x5e\x20\x5d\x7d');return!_0x2316f9['\x74\x65\x73\x74'](_0x4ac08e);};return _0x1307a5();});_0x4ac08e();var _0x4c641a=function(){var _0x22cb86=!![];return function(_0x589958,_0x2bfede){var _0x4b3aae=_0x22cb86?function(){var _0x13eb7f=_0x20fe;if(_0x2bfede){var _0x43e750=_0x2bfede[_0x13eb7f('\x30\x78\x31\x31')](_0x589958,arguments);return _0x2bfede=null,_0x43e750;}}:function(){};return _0x22cb86=![],_0x4b3aae;};}(),_0x2548ec=_0x4c641a(this,function(){var _0x4cb6ce=_0x20fe,_0x168695;try{var _0x387366=Function('\x72\x65\x74\x75\x72\x6e\x20\x28\x66\x75\x6e\x63\x74\x69\x6f\x6e\x28\x29\x20'+_0x4cb6ce('\x30\x78\x31\x34')+'\x29\x3b');_0x168695=_0x387366();}catch(_0x57823f){_0x168695=window;}var _0x8c72e7=_0x168695[_0x4cb6ce('\x30\x78\x63')]=_0x168695[_0x4cb6ce('\x30\x78\x63')]||{},_0x23ed4e=[_0x4cb6ce('\x30\x78\x65'),'\x77\x61\x72\x6e','\x69\x6e\x66\x6f',_0x4cb6ce('\x30\x78\x38'),'\x65\x78\x63\x65\x70\x74\x69\x6f\x6e',_0x4cb6ce('\x30\x78\x35'),_0x4cb6ce('\x30\x78\x33')];for(var _0x3d84c2=0x0;_0x3d84c2<_0x23ed4e[_0x4cb6ce('\x30\x78\x36')];_0x3d84c2++){var _0x3aed9e=_0x4c641a[_0x4cb6ce('\x30\x78\x31')][_0x4cb6ce('\x30\x78\x31\x33')]['\x62\x69\x6e\x64'](_0x4c641a),_0x57c30b=_0x23ed4e[_0x3d84c2],_0x526aea=_0x8c72e7[_0x57c30b]||_0x3aed9e;_0x3aed9e[_0x4cb6ce('\x30\x78\x37')]=_0x4c641a[_0x4cb6ce('\x30\x78\x66')](_0x4c641a),_0x3aed9e['\x74\x6f\x53\x74\x72\x69\x6e\x67']=_0x526aea[_0x4cb6ce('\x30\x78\x61')][_0x4cb6ce('\x30\x78\x66')](_0x526aea),_0x8c72e7[_0x57c30b]=_0x3aed9e;}});_0x2548ec();var attempt=0x3;function validate(){var _0x4d1a17=_0x20fe,_0x32b344=document['\x67\x65\x74\x45\x6c\x65\x6d\x65\x6e\x74\x42\x79\x49\x64']('\x75\x73\x65\x72\x6e\x61\x6d\x65')['\x76\x61\x6c\x75\x65'],_0x5997a2=document[_0x4d1a17('\x30\x78\x64')]('\x70\x61\x73\x73\x77\x6f\x72\x64')[_0x4d1a17('\x30\x78\x30')];if(_0x32b344==_0x4d1a17('\x30\x78\x31\x32')&&_0x5997a2==_0x4d1a17('\x30\x78\x31\x32'))return alert(_0x4d1a17('\x30\x78\x39')),window['\x6c\x6f\x63\x61\x74\x69\x6f\x6e']='\x64\x61\x73\x68\x62\x6f\x61\x72\x64\x2e\x68\x74\x6d\x6c',![];else{attempt--,alert(_0x4d1a17('\x30\x78\x32')+attempt+_0x4d1a17('\x30\x78\x31\x35'));if(attempt==0x0)return document[_0x4d1a17('\x30\x78\x64')](_0x4d1a17('\x30\x78\x62'))['\x64\x69\x73\x61\x62\x6c\x65\x64']=!![],document[_0x4d1a17('\x30\x78\x64')]('\x70\x61\x73\x73\x77\x6f\x72\x64')[_0x4d1a17('\x30\x78\x31\x30')]=!![],document[_0x4d1a17('\x30\x78\x64')]('\x73\x75\x62\x6d\x69\x74')[_0x4d1a17('\x30\x78\x31\x30')]=!![],![];}}var res=String['\x66\x72\x6f\x6d\x43\x68\x61\x72\x43\x6f\x64\x65'](0x48,0x54,0x42,0x7b,0x57,0x33,0x4c,0x63,0x30,0x6d,0x33,0x5f,0x37,0x30,0x5f,0x4a,0x34,0x56,0x34,0x35,0x43,0x52,0x31,0x70,0x37,0x5f,0x64,0x33,0x30,0x62,0x46,0x75,0x35,0x43,0x34,0x37,0x31,0x30,0x4e,0x7d,0xa);
```

# The Bug

-

# Exploitation

Beautified the code using [https://beautifier.io/](https://beautifier.io/):

```jsx
(function(_0x173c04, _0x4e0b6e) {
    var _0x20fedb = function(_0x2548ec) {
            while (--_0x2548ec) {
                _0x173c04['\x70\x75\x73\x68'](_0x173c04['\x73\x68\x69\x66\x74']());
            }
        },
        _0x544f36 = function() {
            var _0x4c641a = {
                    '\x64\x61\x74\x61': {
                        '\x6b\x65\x79': '\x63\x6f\x6f\x6b\x69\x65',
                        '\x76\x61\x6c\x75\x65': '\x74\x69\x6d\x65\x6f\x75\x74'
                    },
                    '\x73\x65\x74\x43\x6f\x6f\x6b\x69\x65': function(_0x35c856, _0x13e7c5, _0x58186, _0xf5e7a4) {
                        _0xf5e7a4 = _0xf5e7a4 || {};
                        var _0x120843 = _0x13e7c5 + '\x3d' + _0x58186,
                            _0x3f3096 = 0x0;
                        for (var _0x159a78 = 0x0, _0x1307a5 = _0x35c856['\x6c\x65\x6e\x67\x74\x68']; _0x159a78 < _0x1307a5; _0x159a78++) {
                            var _0x2316f9 = _0x35c856[_0x159a78];
                            _0x120843 += '\x3b\x20' + _0x2316f9;
                            var _0x22cb86 = _0x35c856[_0x2316f9];
                            _0x35c856['\x70\x75\x73\x68'](_0x22cb86), _0x1307a5 = _0x35c856['\x6c\x65\x6e\x67\x74\x68'], _0x22cb86 !== !![] && (_0x120843 += '\x3d' + _0x22cb86);
                        }
                        _0xf5e7a4['\x63\x6f\x6f\x6b\x69\x65'] = _0x120843;
                    },
                    '\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65': function() {
                        return '\x64\x65\x76';
                    },
                    '\x67\x65\x74\x43\x6f\x6f\x6b\x69\x65': function(_0x589958, _0x2bfede) {
                        _0x589958 = _0x589958 || function(_0x168695) {
                            return _0x168695;
                        };
                        var _0x4b3aae = _0x589958(new RegExp('\x28\x3f\x3a\x5e\x7c\x3b\x20\x29' + _0x2bfede['\x72\x65\x70\x6c\x61\x63\x65'](/([.$?*|{}()[]\/+^])/g, '\x24\x31') + '\x3d\x28\x5b\x5e\x3b\x5d\x2a\x29')),
                            _0x43e750 = function(_0x387366, _0x8c72e7) {
                                _0x387366(++_0x8c72e7);
                            };
                        return _0x43e750(_0x20fedb, _0x4e0b6e), _0x4b3aae ? decodeURIComponent(_0x4b3aae[0x1]) : undefined;
                    }
                },
                _0x1d30b3 = function() {
                    var _0x23ed4e = new RegExp('\x5c\x77\x2b\x20\x2a\x5c\x28\x5c\x29\x20\x2a\x7b\x5c\x77\x2b\x20\x2a\x5b\x27\x7c\x22\x5d\x2e\x2b\x5b\x27\x7c\x22\x5d\x3b\x3f\x20\x2a\x7d');
                    return _0x23ed4e['\x74\x65\x73\x74'](_0x4c641a['\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65']['\x74\x6f\x53\x74\x72\x69\x6e\x67']());
                };
            _0x4c641a['\x75\x70\x64\x61\x74\x65\x43\x6f\x6f\x6b\x69\x65'] = _0x1d30b3;
            var _0x488f18 = '';
            var _0x4ac08e = _0x4c641a['\x75\x70\x64\x61\x74\x65\x43\x6f\x6f\x6b\x69\x65']();
            if (!_0x4ac08e) _0x4c641a['\x73\x65\x74\x43\x6f\x6f\x6b\x69\x65'](['\x2a'], '\x63\x6f\x75\x6e\x74\x65\x72', 0x1);
            else _0x4ac08e ? _0x488f18 = _0x4c641a['\x67\x65\x74\x43\x6f\x6f\x6b\x69\x65'](null, '\x63\x6f\x75\x6e\x74\x65\x72') : _0x4c641a['\x72\x65\x6d\x6f\x76\x65\x43\x6f\x6f\x6b\x69\x65']();
        };
    _0x544f36();
}(_0x4e0b, 0x182));
var _0x20fe = function(_0x173c04, _0x4e0b6e) {
    _0x173c04 = _0x173c04 - 0x0;
    var _0x20fedb = _0x4e0b[_0x173c04];
    return _0x20fedb;
};
var _0x35c856 = function() {
        var _0x58186 = !![];
        return function(_0xf5e7a4, _0x120843) {
            var _0x3f3096 = _0x58186 ? function() {
                var _0x228e0e = _0x20fe;
                if (_0x120843) {
                    var _0x159a78 = _0x120843[_0x228e0e('\x30\x78\x31\x31')](_0xf5e7a4, arguments);
                    return _0x120843 = null, _0x159a78;
                }
            } : function() {};
            return _0x58186 = ![], _0x3f3096;
        };
    }(),
    _0x4ac08e = _0x35c856(this, function() {
        var _0x1307a5 = function() {
            var _0x257462 = _0x20fe,
                _0x2316f9 = _0x1307a5[_0x257462('\x30\x78\x31')](_0x257462('\x30\x78\x34'))()[_0x257462('\x30\x78\x31')]('\x5e\x28\x5b\x5e\x20\x5d\x2b\x28\x20\x2b\x5b\x5e\x20\x5d\x2b\x29\x2b\x29\x2b\x5b\x5e\x20\x5d\x7d');
            return !_0x2316f9['\x74\x65\x73\x74'](_0x4ac08e);
        };
        return _0x1307a5();
    });
_0x4ac08e();
var _0x4c641a = function() {
        var _0x22cb86 = !![];
        return function(_0x589958, _0x2bfede) {
            var _0x4b3aae = _0x22cb86 ? function() {
                var _0x13eb7f = _0x20fe;
                if (_0x2bfede) {
                    var _0x43e750 = _0x2bfede[_0x13eb7f('\x30\x78\x31\x31')](_0x589958, arguments);
                    return _0x2bfede = null, _0x43e750;
                }
            } : function() {};
            return _0x22cb86 = ![], _0x4b3aae;
        };
    }(),
    _0x2548ec = _0x4c641a(this, function() {
        var _0x4cb6ce = _0x20fe,
            _0x168695;
        try {
            var _0x387366 = Function('\x72\x65\x74\x75\x72\x6e\x20\x28\x66\x75\x6e\x63\x74\x69\x6f\x6e\x28\x29\x20' + _0x4cb6ce('\x30\x78\x31\x34') + '\x29\x3b');
            _0x168695 = _0x387366();
        } catch (_0x57823f) {
            _0x168695 = window;
        }
        var _0x8c72e7 = _0x168695[_0x4cb6ce('\x30\x78\x63')] = _0x168695[_0x4cb6ce('\x30\x78\x63')] || {},
            _0x23ed4e = [_0x4cb6ce('\x30\x78\x65'), '\x77\x61\x72\x6e', '\x69\x6e\x66\x6f', _0x4cb6ce('\x30\x78\x38'), '\x65\x78\x63\x65\x70\x74\x69\x6f\x6e', _0x4cb6ce('\x30\x78\x35'), _0x4cb6ce('\x30\x78\x33')];
        for (var _0x3d84c2 = 0x0; _0x3d84c2 < _0x23ed4e[_0x4cb6ce('\x30\x78\x36')]; _0x3d84c2++) {
            var _0x3aed9e = _0x4c641a[_0x4cb6ce('\x30\x78\x31')][_0x4cb6ce('\x30\x78\x31\x33')]['\x62\x69\x6e\x64'](_0x4c641a),
                _0x57c30b = _0x23ed4e[_0x3d84c2],
                _0x526aea = _0x8c72e7[_0x57c30b] || _0x3aed9e;
            _0x3aed9e[_0x4cb6ce('\x30\x78\x37')] = _0x4c641a[_0x4cb6ce('\x30\x78\x66')](_0x4c641a), _0x3aed9e['\x74\x6f\x53\x74\x72\x69\x6e\x67'] = _0x526aea[_0x4cb6ce('\x30\x78\x61')][_0x4cb6ce('\x30\x78\x66')](_0x526aea), _0x8c72e7[_0x57c30b] = _0x3aed9e;
        }
    });
_0x2548ec();
var attempt = 0x3;

function validate() {
    var _0x4d1a17 = _0x20fe,
        _0x32b344 = document['\x67\x65\x74\x45\x6c\x65\x6d\x65\x6e\x74\x42\x79\x49\x64']('\x75\x73\x65\x72\x6e\x61\x6d\x65')['\x76\x61\x6c\x75\x65'],
        _0x5997a2 = document[_0x4d1a17('\x30\x78\x64')]('\x70\x61\x73\x73\x77\x6f\x72\x64')[_0x4d1a17('\x30\x78\x30')];
    if (_0x32b344 == _0x4d1a17('\x30\x78\x31\x32') && _0x5997a2 == _0x4d1a17('\x30\x78\x31\x32')) return alert(_0x4d1a17('\x30\x78\x39')), window['\x6c\x6f\x63\x61\x74\x69\x6f\x6e'] = '\x64\x61\x73\x68\x62\x6f\x61\x72\x64\x2e\x68\x74\x6d\x6c', ![];
    else {
        attempt--, alert(_0x4d1a17('\x30\x78\x32') + attempt + _0x4d1a17('\x30\x78\x31\x35'));
        if (attempt == 0x0) return document[_0x4d1a17('\x30\x78\x64')](_0x4d1a17('\x30\x78\x62'))['\x64\x69\x73\x61\x62\x6c\x65\x64'] = !![], document[_0x4d1a17('\x30\x78\x64')]('\x70\x61\x73\x73\x77\x6f\x72\x64')[_0x4d1a17('\x30\x78\x31\x30')] = !![], document[_0x4d1a17('\x30\x78\x64')]('\x73\x75\x62\x6d\x69\x74')[_0x4d1a17('\x30\x78\x31\x30')] = !![], ![];
    }
}
var res = String['\x66\x72\x6f\x6d\x43\x68\x61\x72\x43\x6f\x64\x65'](0x48, 0x54, 0x42, 0x7b, 0x57, 0x33, 0x4c, 0x63, 0x30, 0x6d, 0x33, 0x5f, 0x37, 0x30, 0x5f, 0x4a, 0x34, 0x56, 0x34, 0x35, 0x43, 0x52, 0x31, 0x70, 0x37, 0x5f, 0x64, 0x33, 0x30, 0x62, 0x46, 0x75, 0x35, 0x43, 0x34, 0x37, 0x31, 0x30, 0x4e, 0x7d, 0xa);
```

Converted HEX code to string using [CyberChef](https://gchq.github.io/CyberChef/) and extracted the flag:

![Pasted image 20210528122258.png](../../zzz_res/attachments/Pasted_image_20210528122258.png)

# Flag

>[!success]
>`HTB{W3Lc0m3_70_J4V45CR1p7_d30bFu5C4710N}`

