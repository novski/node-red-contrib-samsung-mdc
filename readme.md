# Samsung MDC Protocol
available [online](https://aca.im/driver_docs/Samsung/MDC%20Protocol%202015%20v13.7c.pdf) over different providers. Seams like Samsung does not provide it directly.

## How to use
### Input
 - Set Host, Port, MAC and ID of the device in the environement variables of the subflow node (click on the i symbol)
 - Give in a Command as msg.topic String `"0xYY"` any Hexcode declared in the Protocol.
 - Declare any Value targeted as Data1.. in the Protocol as Payload String `"0xYY"`or Array of Strings.

### Output
 - Expect the full response as msg.payload.response.
 - id as msg.payload.id
 - command as msg.payload.command
 - All Values get exponated in msg.payload.response.values[]

## Specialites
Power needs to be triggerd as Wake on Lan 3 times within 2s if the display is in Power Off status. This is done inside the subnode regardless the status anytime a `NAK` after power on (`0x11`) arrives.


Example flow:
`[
    {
        "id": "39374379.e31f4c",
        "type": "subflow",
        "name": "Samsung MDC (2)",
        "info": "# Samsung MDC Protocol\navailable [online](https://aca.im/driver_docs/Samsung/MDC%20Protocol%202015%20v13.7c.pdf).\n\n## how to use\n### input\n - Set Host, Port, MAC and ID of the device in the environement variables of the subflow node (click on the i symbol)\n - Give in a Command as msg.topic String `\"0xYY\"` any Hexcode declared in the Protocol.\n - Declare any Value targeted as Data1.. in the Protocol as Payload String `\"0xYY\"`or Array of Strings.\n\n### output\n - Expect the full response as msg.payload.response.\n - id as msg.payload.id\n - command as msg.payload.command\n - All Values get exponated in msg.payload.response.values[]\n\n## Specialites\nPower needs to be triggerd as Wake on Lan 3 times within 2s if the display is in Power Off status. This is done inside the subnode regardless the status anytime a `NAK` after power on (`0x11`) arrives.\n\n\nExample flow:\n```[\n    {\n        \"id\": \"721b0fbe.8c39e\",\n        \"type\": \"subflow\",\n        \"name\": \"Samsung MDC\",\n        \"info\": \"# Samsung MDC Protocol\\navailable [online](https://aca.im/driver_docs/Samsung/MDC%20Protocol%202015%20v13.7c.pdf).\\n\\n## how to use\\n### input\\n - Set Host, Port, MAC and ID of the device in the environement variables of the subflow node (click on the i symbol)\\n - Give in a Command as msg.topic String `\\\"0xYY\\\"` any Hexcode declared in the Protocol.\\n - Declare any Value targeted as Data1.. in the Protocol as Payload String `\\\"0xYY\\\"`or Array of Strings.\\n\\n### output\\n - Expect the full response as msg.payload.response.\\n - id as msg.payload.id\\n - command as msg.payload.command\\n - All Values get exponated in msg.payload.response.values[]\\n\\n## Specialites\\nPower needs to be triggerd as Wake on Lan 3 times within 2s if the display is in Power Off status. This is done inside the subnode regardless the status anytime a `NAK` after power on (`0x11`) arrives.\",\n        \"category\": \"\",\n        \"in\": [\n            {\n                \"x\": 60,\n                \"y\": 100,\n                \"wires\": [\n                    {\n                        \"id\": \"5f07fe4e.7b1b4\"\n                    }\n                ]\n            }\n        ],\n        \"out\": [\n            {\n                \"x\": 2700,\n                \"y\": 40,\n                \"wires\": [\n                    {\n                        \"id\": \"ca2d05c8.1f9178\",\n                        \"port\": 0\n                    }\n                ]\n            },\n            {\n                \"x\": 2700,\n                \"y\": 80,\n                \"wires\": [\n                    {\n                        \"id\": \"32fd9576.38cdaa\",\n                        \"port\": 0\n                    }\n                ]\n            }\n        ],\n        \"env\": [\n            {\n                \"name\": \"MAC\",\n                \"type\": \"str\",\n                \"value\": \"\"\n            },\n            {\n                \"name\": \"Host\",\n                \"type\": \"str\",\n                \"value\": \"\"\n            },\n            {\n                \"name\": \"Port\",\n                \"type\": \"num\",\n                \"value\": \"1515\"\n            },\n            {\n                \"name\": \"Screen ID\",\n                \"type\": \"num\",\n                \"value\": \"\"\n            }\n        ],\n        \"color\": \"#3FADB5\",\n        \"outputLabels\": [\n            \"response\",\n            \"errorcode\"\n        ],\n        \"icon\": \"node-red-node-wol/onoff.png\",\n        \"status\": {\n            \"x\": 2700,\n            \"y\": 160,\n            \"wires\": [\n                {\n                    \"id\": \"19ea9eee.a224c1\",\n                    \"port\": 0\n                },\n                {\n                    \"id\": \"32fd9576.38cdaa\",\n                    \"port\": 0\n                },\n                {\n                    \"id\": \"1b4f5b0e.11c705\",\n                    \"port\": 0\n                }\n            ]\n        }\n    },\n    {\n        \"id\": \"5483af31.0ea2d\",\n        \"type\": \"function\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"request creation\",\n        \"func\": \"// MDC protocol:     Header,Command, ID,   Datalength, Data1, Checksum\\n// Example Power ON: 0xAA,  0x11,    0x01, 0x01,       0x01,  0x14]);\\n\\nconst header = 0xAA;\\nlet command = parseInt(msg.topic,16);\\nlet id = parseInt(msg.id,16);\\nlet message = [];\\nmessage.push(header, command, id)\\nlet data_sum = 0;\\nlet datalength = 0;\\nlet value = msg.payload;\\nif (value === false){\\n    message.push(datalength);\\n} else if (Array.isArray(value)){\\n    datalength = value.length\\n    value.forEach(function(data){\\n        let d = parseInt(data,16)\\n        message.push(datalength,d);\\n        data_sum += d;\\n    });\\n} else {\\n    datalength = 1\\n    let d = parseInt(value,16)\\n    message.push(datalength,d);\\n    data_sum += d;\\n}\\nlet checksum = command + id + datalength + data_sum;\\nmessage.push(checksum);\\nmsg.payload = new Buffer(message);\\nmsg.request = {\\n    'host':msg.host,\\n    'port':msg.port,\\n    'id':msg.id,\\n    'topic':msg.topic,\\n    'value':value,\\n    'payload':msg.payload\\n}\\nreturn msg;\",\n        \"outputs\": 1,\n        \"noerr\": 0,\n        \"initialize\": \"\",\n        \"finalize\": \"\",\n        \"x\": 700,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"30335085.8ba44\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"2d697c27.744534\",\n        \"type\": \"wake on lan\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"mac\": \"\",\n        \"host\": \"\",\n        \"udpport\": 9,\n        \"name\": \"\",\n        \"x\": 2650,\n        \"y\": 260,\n        \"wires\": []\n    },\n    {\n        \"id\": \"5f07fe4e.7b1b4\",\n        \"type\": \"function\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"if (topic) isHex:true|false\",\n        \"func\": \"function isHex(h) {\\n    var a = parseInt(h,16);\\n    if (a.toString(16).length === 1){\\n        return (('0x0'+a.toString(16)) === h.toLowerCase())\\n    } else {\\n        return (('0x'+a.toString(16)) === h.toLowerCase())\\n    }\\n}\\nif (isHex(msg.topic)){\\n    return [msg,null]\\n}else{\\n    return [null, msg]\\n}\",\n        \"outputs\": 2,\n        \"noerr\": 0,\n        \"initialize\": \"\",\n        \"finalize\": \"\",\n        \"x\": 230,\n        \"y\": 100,\n        \"wires\": [\n            [\n                \"b60b67eb.8b5368\"\n            ],\n            [\n                \"19ea9eee.a224c1\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"30335085.8ba44\",\n        \"type\": \"tcp request\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"server\": \"\",\n        \"port\": \"\",\n        \"out\": \"time\",\n        \"splitc\": \"1900\",\n        \"name\": \"\",\n        \"x\": 890,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"ebe4e7c6.32f028\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"19ea9eee.a224c1\",\n        \"type\": \"change\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"error not valid hex topic\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"Topic not a valid hex number. \\\\n Declare as type string and 0x00 format. \",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 510,\n        \"y\": 140,\n        \"wires\": [\n            []\n        ]\n    },\n    {\n        \"id\": \"ca2d05c8.1f9178\",\n        \"type\": \"function\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"response\",\n        \"func\": \"msg.response = {};\\nif (msg.payload[5].toString(16).length === 1){\\n    msg.response.command = '0x0'+msg.payload[5].toString(16);\\n} else {\\n    msg.response.command = '0x'+msg.payload[5].toString(16);\\n}\\nmsg.response.count = (parseInt(msg.payload[3],16)-2);\\nmsg.response.val = []\\nfor(let i = 1; i <= msg.response.count; i++){\\n    index = 5 + i\\n    if (msg.payload[index].toString(16).length === 1){\\n        msg.response.val[i] = '0x0'+msg.payload[index].toString(16)\\n    } else {\\n        msg.response.val[i] = '0x'+msg.payload[index].toString(16)\\n    }\\n}\\nreturn msg;\",\n        \"outputs\": 1,\n        \"noerr\": 0,\n        \"initialize\": \"\",\n        \"finalize\": \"\",\n        \"x\": 1860,\n        \"y\": 40,\n        \"wires\": [\n            []\n        ]\n    },\n    {\n        \"id\": \"a63d8c50.2d0f5\",\n        \"type\": \"switch\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"response command 0xFF\",\n        \"property\": \"payload[1]\",\n        \"propertyType\": \"msg\",\n        \"rules\": [\n            {\n                \"t\": \"eq\",\n                \"v\": \"255\",\n                \"vt\": \"num\"\n            }\n        ],\n        \"checkall\": \"true\",\n        \"repair\": false,\n        \"outputs\": 1,\n        \"x\": 1330,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"2e75b8b8.ae0c98\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"2e75b8b8.ae0c98\",\n        \"type\": \"switch\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"response command ACK|NAK\",\n        \"property\": \"payload[4]\",\n        \"propertyType\": \"msg\",\n        \"rules\": [\n            {\n                \"t\": \"eq\",\n                \"v\": \"65\",\n                \"vt\": \"num\"\n            },\n            {\n                \"t\": \"eq\",\n                \"v\": \"78\",\n                \"vt\": \"num\"\n            }\n        ],\n        \"checkall\": \"true\",\n        \"repair\": false,\n        \"outputs\": 2,\n        \"x\": 1610,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"ca2d05c8.1f9178\"\n            ],\n            [\n                \"8bbd972e.dfaba8\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"32fd9576.38cdaa\",\n        \"type\": \"change\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"errorcode\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"\\\"error occurred, code:\\\"+$String($formatBase(payload[6],16))\",\n                \"tot\": \"jsonata\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"response.cmd\",\n                \"pt\": \"msg\",\n                \"to\": \"$formatBase(payload[5],16)\",\n                \"tot\": \"jsonata\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"response.val[1]\",\n                \"pt\": \"msg\",\n                \"to\": \"$formatBase(payload[6],16)\",\n                \"tot\": \"jsonata\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 2160,\n        \"y\": 80,\n        \"wires\": [\n            []\n        ]\n    },\n    {\n        \"id\": \"8bbd972e.dfaba8\",\n        \"type\": \"switch\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"if 0x11 (17) power fail\",\n        \"property\": \"payload[5]\",\n        \"propertyType\": \"msg\",\n        \"rules\": [\n            {\n                \"t\": \"else\"\n            },\n            {\n                \"t\": \"eq\",\n                \"v\": \"17\",\n                \"vt\": \"num\"\n            }\n        ],\n        \"checkall\": \"true\",\n        \"repair\": false,\n        \"outputs\": 2,\n        \"x\": 1900,\n        \"y\": 80,\n        \"wires\": [\n            [\n                \"32fd9576.38cdaa\"\n            ],\n            [\n                \"1c0a0b4b.78a115\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"1c0a0b4b.78a115\",\n        \"type\": \"change\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"flow.power_fail + 1\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"power_fail\",\n                \"pt\": \"flow\",\n                \"to\": \"$flowContext('power_fail') + 1\",\n                \"tot\": \"jsonata\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"mac\",\n                \"pt\": \"msg\",\n                \"to\": \"MAC\",\n                \"tot\": \"env\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"host\",\n                \"pt\": \"msg\",\n                \"to\": \"Host\",\n                \"tot\": \"env\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"id\",\n                \"pt\": \"msg\",\n                \"to\": \"Screen ID\",\n                \"tot\": \"env\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x11\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 2190,\n        \"y\": 260,\n        \"wires\": [\n            [\n                \"665519e4.5f4798\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"665519e4.5f4798\",\n        \"type\": \"switch\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"flow.power_fail <= 3\",\n        \"property\": \"power_fail\",\n        \"propertyType\": \"flow\",\n        \"rules\": [\n            {\n                \"t\": \"lte\",\n                \"v\": \"3\",\n                \"vt\": \"num\"\n            }\n        ],\n        \"checkall\": \"true\",\n        \"repair\": false,\n        \"outputs\": 1,\n        \"x\": 2430,\n        \"y\": 260,\n        \"wires\": [\n            [\n                \"2d697c27.744534\",\n                \"25fef0c0.1dbf5\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"25fef0c0.1dbf5\",\n        \"type\": \"delay\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"\",\n        \"pauseType\": \"delay\",\n        \"timeout\": \"2\",\n        \"timeoutUnits\": \"seconds\",\n        \"rate\": \"1\",\n        \"nbRateUnits\": \"1\",\n        \"rateUnits\": \"second\",\n        \"randomFirst\": \"1\",\n        \"randomLast\": \"5\",\n        \"randomUnits\": \"seconds\",\n        \"drop\": false,\n        \"x\": 2600,\n        \"y\": 320,\n        \"wires\": [\n            [\n                \"e4bf22ee.b139b\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"b60b67eb.8b5368\",\n        \"type\": \"change\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"set host, port & ID \",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"host\",\n                \"pt\": \"msg\",\n                \"to\": \"Host\",\n                \"tot\": \"env\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"port\",\n                \"pt\": \"msg\",\n                \"to\": \"Port\",\n                \"tot\": \"env\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"id\",\n                \"pt\": \"msg\",\n                \"to\": \"Screen ID\",\n                \"tot\": \"env\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 490,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"5483af31.0ea2d\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"e4bf22ee.b139b\",\n        \"type\": \"link out\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"\",\n        \"links\": [\n            \"15c3028f.d91acd\"\n        ],\n        \"x\": 2695,\n        \"y\": 320,\n        \"wires\": []\n    },\n    {\n        \"id\": \"15c3028f.d91acd\",\n        \"type\": \"link in\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"\",\n        \"links\": [\n            \"e4bf22ee.b139b\"\n        ],\n        \"x\": 335,\n        \"y\": 40,\n        \"wires\": [\n            [\n                \"b60b67eb.8b5368\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"ebe4e7c6.32f028\",\n        \"type\": \"change\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"del: topic, host, port, id\",\n        \"rules\": [\n            {\n                \"t\": \"delete\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\"\n            },\n            {\n                \"t\": \"delete\",\n                \"p\": \"host\",\n                \"pt\": \"msg\"\n            },\n            {\n                \"t\": \"delete\",\n                \"p\": \"port\",\n                \"pt\": \"msg\"\n            },\n            {\n                \"t\": \"delete\",\n                \"p\": \"id\",\n                \"pt\": \"msg\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1090,\n        \"y\": 60,\n        \"wires\": [\n            [\n                \"a63d8c50.2d0f5\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"1b4f5b0e.11c705\",\n        \"type\": \"status\",\n        \"z\": \"721b0fbe.8c39e\",\n        \"name\": \"\",\n        \"scope\": null,\n        \"x\": 2580,\n        \"y\": 200,\n        \"wires\": [\n            []\n        ]\n    },\n    {\n        \"id\": \"3e444505.2b2e6a\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: Power Control, payload:on\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x11\",\n                \"tot\": \"str\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"0x01\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1120,\n        \"y\": 4420,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"53ea2ab2.a35174\",\n        \"type\": \"debug\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"resp\",\n        \"active\": true,\n        \"tosidebar\": true,\n        \"console\": false,\n        \"tostatus\": false,\n        \"complete\": \"true\",\n        \"targetType\": \"full\",\n        \"statusVal\": \"\",\n        \"statusType\": \"auto\",\n        \"x\": 1810,\n        \"y\": 4460,\n        \"wires\": []\n    },\n    {\n        \"id\": \"f11fe539.7258a8\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"ON\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"true\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4420,\n        \"wires\": [\n            [\n                \"3e444505.2b2e6a\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"9afab070.8362c\",\n        \"type\": \"subflow:721b0fbe.8c39e\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"Samsung MDC\",\n        \"env\": [\n            {\n                \"name\": \"MAC\",\n                \"value\": \"00:19:32:01:1f:77\",\n                \"type\": \"str\"\n            },\n            {\n                \"name\": \"Host\",\n                \"value\": \"16-VMON-1\",\n                \"type\": \"str\"\n            },\n            {\n                \"name\": \"Screen ID\",\n                \"value\": \"1\",\n                \"type\": \"num\"\n            }\n        ],\n        \"x\": 1440,\n        \"y\": 4520,\n        \"wires\": [\n            [\n                \"388e8eff.666892\"\n            ],\n            [\n                \"f4b1433e.b396\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"f4b1433e.b396\",\n        \"type\": \"debug\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"err\",\n        \"active\": true,\n        \"tosidebar\": true,\n        \"console\": false,\n        \"tostatus\": false,\n        \"complete\": \"true\",\n        \"targetType\": \"full\",\n        \"statusVal\": \"\",\n        \"statusType\": \"auto\",\n        \"x\": 1650,\n        \"y\": 4580,\n        \"wires\": []\n    },\n    {\n        \"id\": \"377df4b3.34e3cc\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: Power Control, payload:off\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x11\",\n                \"tot\": \"str\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"0x00\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1120,\n        \"y\": 4460,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"f6b8e26a.f1af3\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"OFF\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"false\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4460,\n        \"wires\": [\n            [\n                \"377df4b3.34e3cc\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"15a6ec7c.f17db4\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: source, payload:HDMI1\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x14\",\n                \"tot\": \"str\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"0x21\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1110,\n        \"y\": 4500,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"2cf9d09b.28e5\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"trigger\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"false\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4500,\n        \"wires\": [\n            [\n                \"15a6ec7c.f17db4\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"3c97fb1c.7d3e24\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: source, payload:HDMI2\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x14\",\n                \"tot\": \"str\"\n            },\n            {\n                \"t\": \"set\",\n                \"p\": \"payload\",\n                \"pt\": \"msg\",\n                \"to\": \"0x23\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1110,\n        \"y\": 4540,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"3f8a932d.368bbc\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"trigger\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"false\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4540,\n        \"wires\": [\n            [\n                \"3c97fb1c.7d3e24\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"c5257b44.ca7688\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: GET status\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x00\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1070,\n        \"y\": 4580,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"48dc33ef.6c661c\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"trigger\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"false\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4580,\n        \"wires\": [\n            [\n                \"c5257b44.ca7688\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"6361f50e.d395ac\",\n        \"type\": \"change\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"topic: GET Display status\",\n        \"rules\": [\n            {\n                \"t\": \"set\",\n                \"p\": \"topic\",\n                \"pt\": \"msg\",\n                \"to\": \"0x0d\",\n                \"tot\": \"str\"\n            }\n        ],\n        \"action\": \"\",\n        \"property\": \"\",\n        \"from\": \"\",\n        \"to\": \"\",\n        \"reg\": false,\n        \"x\": 1090,\n        \"y\": 4620,\n        \"wires\": [\n            [\n                \"9afab070.8362c\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"19dc34d6.84ec4b\",\n        \"type\": \"inject\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"trigger\",\n        \"props\": [\n            {\n                \"p\": \"payload\"\n            }\n        ],\n        \"repeat\": \"\",\n        \"crontab\": \"\",\n        \"once\": false,\n        \"onceDelay\": 0.1,\n        \"topic\": \"\",\n        \"payload\": \"false\",\n        \"payloadType\": \"bool\",\n        \"x\": 910,\n        \"y\": 4620,\n        \"wires\": [\n            [\n                \"6361f50e.d395ac\"\n            ]\n        ]\n    },\n    {\n        \"id\": \"388e8eff.666892\",\n        \"type\": \"function\",\n        \"z\": \"61a52bac.14f714\",\n        \"name\": \"\",\n        \"func\": \"if(msg.response.command === '0x0d'){\\n    msg.temperature_C = parseInt(msg.response.val[5])\\n} else if (msg.response.command === '0x00'){\\n    msg.power = parseInt(msg.response.val[1])\\n    msg.volume = parseInt(msg.response.val[2])\\n    msg.mute = parseInt(msg.response.val[3])\\n    if (msg.response.val[4] === \\\"0x21\\\"){\\n        msg.in = 'HDMI1'\\n    } else if (msg.response.val[4] === \\\"0x23\\\"){\\n        msg.in = 'HDMI2'\\n    } \\n    msg.aspect = parseInt(msg.response.val[5])\\n}\\nreturn msg;\",\n        \"outputs\": 1,\n        \"noerr\": 0,\n        \"initialize\": \"\",\n        \"finalize\": \"\",\n        \"x\": 1660,\n        \"y\": 4460,\n        \"wires\": [\n            [\n                \"53ea2ab2.a35174\"\n            ]\n        ]\n    }\n]```",
        "category": "",
        "in": [
            {
                "x": 60,
                "y": 100,
                "wires": [
                    {
                        "id": "620fa9a0.6290e8"
                    }
                ]
            }
        ],
        "out": [
            {
                "x": 2700,
                "y": 40,
                "wires": [
                    {
                        "id": "6850b007.741de",
                        "port": 0
                    }
                ]
            },
            {
                "x": 2700,
                "y": 80,
                "wires": [
                    {
                        "id": "b0067c54.3563e",
                        "port": 0
                    }
                ]
            }
        ],
        "env": [
            {
                "name": "Broadcast IP",
                "type": "str",
                "value": ""
            },
            {
                "name": "Broadcast port",
                "type": "num",
                "value": "9"
            },
            {
                "name": "MAC",
                "type": "str",
                "value": ""
            },
            {
                "name": "Host",
                "type": "str",
                "value": ""
            },
            {
                "name": "Port",
                "type": "num",
                "value": "1515"
            },
            {
                "name": "Screen ID",
                "type": "num",
                "value": ""
            }
        ],
        "color": "#3FADB5",
        "outputLabels": [
            "response",
            "errorcode"
        ],
        "icon": "node-red-node-wol/onoff.png",
        "status": {
            "x": 2700,
            "y": 160,
            "wires": [
                {
                    "id": "90abec12.d2034",
                    "port": 0
                },
                {
                    "id": "b0067c54.3563e",
                    "port": 0
                },
                {
                    "id": "fcbb6a62.7c88b8",
                    "port": 0
                }
            ]
        }
    },
    {
        "id": "459752a4.5e8c2c",
        "type": "function",
        "z": "39374379.e31f4c",
        "name": "request creation",
        "func": "// MDC protocol:     Header,Command, ID,   Datalength, Data1, Checksum\n// Example Power ON: 0xAA,  0x11,    0x01, 0x01,       0x01,  0x14]);\n\nconst header = 0xAA;\nlet command = parseInt(msg.topic,16);\nlet id = parseInt(msg.id,16);\nlet message = [];\nmessage.push(header, command, id)\nlet data_sum = 0;\nlet datalength = 0;\nlet value = msg.payload;\nif (value === false){\n    message.push(datalength);\n} else if (Array.isArray(value)){\n    datalength = value.length\n    value.forEach(function(data){\n        let d = parseInt(data,16)\n        message.push(datalength,d);\n        data_sum += d;\n    });\n} else {\n    datalength = 1\n    let d = parseInt(value,16)\n    message.push(datalength,d);\n    data_sum += d;\n}\nlet checksum = command + id + datalength + data_sum;\nmessage.push(checksum);\nmsg.payload = new Buffer(message);\nmsg.request = {\n    'host':msg.host,\n    'port':msg.port,\n    'id':msg.id,\n    'topic':msg.topic,\n    'value':value,\n    'payload':msg.payload\n}\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "x": 700,
        "y": 60,
        "wires": [
            [
                "fb955ee3.c50a6"
            ]
        ]
    },
    {
        "id": "e87afdbf.69597",
        "type": "wake on lan",
        "z": "39374379.e31f4c",
        "mac": "",
        "host": "",
        "udpport": 9,
        "name": "",
        "x": 2650,
        "y": 260,
        "wires": []
    },
    {
        "id": "620fa9a0.6290e8",
        "type": "function",
        "z": "39374379.e31f4c",
        "name": "if (topic) isHex:true|false",
        "func": "function isHex(h) {\n    var a = parseInt(h,16);\n    if (a.toString(16).length === 1){\n        return (('0x0'+a.toString(16)) === h.toLowerCase())\n    } else {\n        return (('0x'+a.toString(16)) === h.toLowerCase())\n    }\n}\nif (isHex(msg.topic)){\n    return [msg,null]\n}else{\n    return [null, msg]\n}",
        "outputs": 2,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "x": 230,
        "y": 100,
        "wires": [
            [
                "1b9f25ca.969b2a"
            ],
            [
                "90abec12.d2034"
            ]
        ]
    },
    {
        "id": "fb955ee3.c50a6",
        "type": "tcp request",
        "z": "39374379.e31f4c",
        "server": "",
        "port": "",
        "out": "time",
        "splitc": "1900",
        "name": "",
        "x": 890,
        "y": 60,
        "wires": [
            [
                "48025eb6.a1693"
            ]
        ]
    },
    {
        "id": "90abec12.d2034",
        "type": "change",
        "z": "39374379.e31f4c",
        "name": "error not valid hex topic",
        "rules": [
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "Topic not a valid hex number. \\n Declare as type string and 0x00 format. ",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 510,
        "y": 140,
        "wires": [
            []
        ]
    },
    {
        "id": "6850b007.741de",
        "type": "function",
        "z": "39374379.e31f4c",
        "name": "response",
        "func": "msg.response = {};\nif (msg.payload[5].toString(16).length === 1){\n    msg.response.command = '0x0'+msg.payload[5].toString(16);\n} else {\n    msg.response.command = '0x'+msg.payload[5].toString(16);\n}\nmsg.response.count = (parseInt(msg.payload[3],16)-2);\nmsg.response.val = []\nfor(let i = 1; i <= msg.response.count; i++){\n    index = 5 + i\n    if (msg.payload[index].toString(16).length === 1){\n        msg.response.val[i] = '0x0'+msg.payload[index].toString(16)\n    } else {\n        msg.response.val[i] = '0x'+msg.payload[index].toString(16)\n    }\n}\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "x": 1860,
        "y": 40,
        "wires": [
            []
        ]
    },
    {
        "id": "ceb58a6c.ce9ea8",
        "type": "switch",
        "z": "39374379.e31f4c",
        "name": "response command 0xFF",
        "property": "payload[1]",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "255",
                "vt": "num"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 1330,
        "y": 60,
        "wires": [
            [
                "57410fe3.f18c8"
            ]
        ]
    },
    {
        "id": "57410fe3.f18c8",
        "type": "switch",
        "z": "39374379.e31f4c",
        "name": "response command ACK|NAK",
        "property": "payload[4]",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "65",
                "vt": "num"
            },
            {
                "t": "eq",
                "v": "78",
                "vt": "num"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 1610,
        "y": 60,
        "wires": [
            [
                "6850b007.741de"
            ],
            [
                "25a19386.a785dc"
            ]
        ]
    },
    {
        "id": "b0067c54.3563e",
        "type": "change",
        "z": "39374379.e31f4c",
        "name": "errorcode",
        "rules": [
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "\"error occurred, code:\"+$String($formatBase(payload[6],16))",
                "tot": "jsonata"
            },
            {
                "t": "set",
                "p": "response.cmd",
                "pt": "msg",
                "to": "$formatBase(payload[5],16)",
                "tot": "jsonata"
            },
            {
                "t": "set",
                "p": "response.val[1]",
                "pt": "msg",
                "to": "$formatBase(payload[6],16)",
                "tot": "jsonata"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 2160,
        "y": 80,
        "wires": [
            []
        ]
    },
    {
        "id": "25a19386.a785dc",
        "type": "switch",
        "z": "39374379.e31f4c",
        "name": "if 0x11 (17) power fail",
        "property": "payload[5]",
        "propertyType": "msg",
        "rules": [
            {
                "t": "else"
            },
            {
                "t": "eq",
                "v": "17",
                "vt": "num"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 1900,
        "y": 80,
        "wires": [
            [
                "b0067c54.3563e"
            ],
            [
                "fd5572d1.e85a4"
            ]
        ]
    },
    {
        "id": "fd5572d1.e85a4",
        "type": "change",
        "z": "39374379.e31f4c",
        "name": "flow.power_fail + 1",
        "rules": [
            {
                "t": "set",
                "p": "power_fail",
                "pt": "flow",
                "to": "$flowContext('power_fail') + 1",
                "tot": "jsonata"
            },
            {
                "t": "set",
                "p": "mac",
                "pt": "msg",
                "to": "MAC",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "host",
                "pt": "msg",
                "to": "Broadcast IP",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "udpport",
                "pt": "msg",
                "to": "Broadcast port",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "id",
                "pt": "msg",
                "to": "Screen ID",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x11",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 2190,
        "y": 260,
        "wires": [
            [
                "69d7e362.dd04fc"
            ]
        ]
    },
    {
        "id": "69d7e362.dd04fc",
        "type": "switch",
        "z": "39374379.e31f4c",
        "name": "flow.power_fail <= 3",
        "property": "power_fail",
        "propertyType": "flow",
        "rules": [
            {
                "t": "lte",
                "v": "3",
                "vt": "num"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 2430,
        "y": 260,
        "wires": [
            [
                "e87afdbf.69597",
                "9691a404.906e28"
            ]
        ]
    },
    {
        "id": "9691a404.906e28",
        "type": "delay",
        "z": "39374379.e31f4c",
        "name": "",
        "pauseType": "delay",
        "timeout": "2",
        "timeoutUnits": "seconds",
        "rate": "1",
        "nbRateUnits": "1",
        "rateUnits": "second",
        "randomFirst": "1",
        "randomLast": "5",
        "randomUnits": "seconds",
        "drop": false,
        "x": 2600,
        "y": 320,
        "wires": [
            [
                "97731235.6437e"
            ]
        ]
    },
    {
        "id": "1b9f25ca.969b2a",
        "type": "change",
        "z": "39374379.e31f4c",
        "name": "set host, port & ID ",
        "rules": [
            {
                "t": "set",
                "p": "host",
                "pt": "msg",
                "to": "Host",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "port",
                "pt": "msg",
                "to": "Port",
                "tot": "env"
            },
            {
                "t": "set",
                "p": "id",
                "pt": "msg",
                "to": "Screen ID",
                "tot": "env"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 490,
        "y": 60,
        "wires": [
            [
                "459752a4.5e8c2c"
            ]
        ]
    },
    {
        "id": "97731235.6437e",
        "type": "link out",
        "z": "39374379.e31f4c",
        "name": "",
        "links": [
            "8585f013.4d7d8"
        ],
        "x": 2695,
        "y": 320,
        "wires": []
    },
    {
        "id": "8585f013.4d7d8",
        "type": "link in",
        "z": "39374379.e31f4c",
        "name": "",
        "links": [
            "97731235.6437e"
        ],
        "x": 335,
        "y": 40,
        "wires": [
            [
                "1b9f25ca.969b2a"
            ]
        ]
    },
    {
        "id": "48025eb6.a1693",
        "type": "change",
        "z": "39374379.e31f4c",
        "name": "del: topic, host, port, id",
        "rules": [
            {
                "t": "delete",
                "p": "topic",
                "pt": "msg"
            },
            {
                "t": "delete",
                "p": "host",
                "pt": "msg"
            },
            {
                "t": "delete",
                "p": "port",
                "pt": "msg"
            },
            {
                "t": "delete",
                "p": "id",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 1090,
        "y": 60,
        "wires": [
            [
                "ceb58a6c.ce9ea8"
            ]
        ]
    },
    {
        "id": "fcbb6a62.7c88b8",
        "type": "status",
        "z": "39374379.e31f4c",
        "name": "",
        "scope": null,
        "x": 2580,
        "y": 200,
        "wires": [
            []
        ]
    },
    {
        "id": "9807ab9e.427e98",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: Power Control, payload:on",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x11",
                "tot": "str"
            },
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "0x01",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 460,
        "y": 800,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "e20b5749.16ce88",
        "type": "debug",
        "z": "763b9945.a3a2e8",
        "name": "response",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "statusVal": "",
        "statusType": "auto",
        "x": 1200,
        "y": 880,
        "wires": []
    },
    {
        "id": "6805fe24.aa799",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "ON",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "true",
        "payloadType": "bool",
        "x": 250,
        "y": 800,
        "wires": [
            [
                "9807ab9e.427e98"
            ]
        ]
    },
    {
        "id": "212d9b6a.d2b944",
        "type": "debug",
        "z": "763b9945.a3a2e8",
        "name": "error",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "true",
        "targetType": "full",
        "statusVal": "",
        "statusType": "auto",
        "x": 990,
        "y": 920,
        "wires": []
    },
    {
        "id": "64b72571.3161dc",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: Power Control, payload:off",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x11",
                "tot": "str"
            },
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "0x00",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 460,
        "y": 840,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "2f1734d5.ec38fc",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "OFF",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "false",
        "payloadType": "bool",
        "x": 250,
        "y": 840,
        "wires": [
            [
                "64b72571.3161dc"
            ]
        ]
    },
    {
        "id": "bfc329e9.0677f8",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: source, payload:HDMI1",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x14",
                "tot": "str"
            },
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "0x21",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 450,
        "y": 880,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "75fcdbca.db2084",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "HDMI1",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "false",
        "payloadType": "bool",
        "x": 250,
        "y": 880,
        "wires": [
            [
                "bfc329e9.0677f8"
            ]
        ]
    },
    {
        "id": "709a1775.da6688",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: source, payload:HDMI2",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x14",
                "tot": "str"
            },
            {
                "t": "set",
                "p": "payload",
                "pt": "msg",
                "to": "0x23",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 450,
        "y": 920,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "2f2c610.251d6a",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "HDMI2",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "false",
        "payloadType": "bool",
        "x": 250,
        "y": 920,
        "wires": [
            [
                "709a1775.da6688"
            ]
        ]
    },
    {
        "id": "c5b2eb6f.ee5358",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: GET status",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x00",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 470,
        "y": 960,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "ac9c64d8.0bb7d8",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "Software status",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "false",
        "payloadType": "bool",
        "x": 280,
        "y": 960,
        "wires": [
            [
                "c5b2eb6f.ee5358"
            ]
        ]
    },
    {
        "id": "ab0f5f84.98b9a",
        "type": "change",
        "z": "763b9945.a3a2e8",
        "name": "topic: GET Display status",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "0x0d",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 490,
        "y": 1000,
        "wires": [
            [
                "2716b928.389866"
            ]
        ]
    },
    {
        "id": "e02a31cf.91e8c",
        "type": "inject",
        "z": "763b9945.a3a2e8",
        "name": "Hardware status",
        "props": [
            {
                "p": "payload"
            }
        ],
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "topic": "",
        "payload": "false",
        "payloadType": "bool",
        "x": 280,
        "y": 1000,
        "wires": [
            [
                "ab0f5f84.98b9a"
            ]
        ]
    },
    {
        "id": "2716b928.389866",
        "type": "subflow:39374379.e31f4c",
        "z": "763b9945.a3a2e8",
        "name": "Samsung MDC",
        "env": [
            {
                "name": "Screen ID",
                "value": "0",
                "type": "num"
            }
        ],
        "x": 760,
        "y": 900,
        "wires": [
            [
                "80bfbab6.a34898"
            ],
            [
                "212d9b6a.d2b944"
            ]
        ]
    },
    {
        "id": "80bfbab6.a34898",
        "type": "function",
        "z": "763b9945.a3a2e8",
        "name": "sanitize response",
        "func": "if (msg.response.command === '0x11'){\n    if (msg.response.val[1] === \"0x01\"){\n        msg.power = \"ON\"\n    } else if (msg.response.val[1] === \"0x00\"){\n        msg.power = \"OFF\"\n    }\n} else if (msg.response.command === '0x14'){\n    if (msg.response.val[1] === \"0x21\"){\n        msg.in = 'HDMI1'\n    } else if (msg.response.val[1] === \"0x23\"){\n        msg.in = 'HDMI2'\n    }\n} else if(msg.response.command === '0x0d'){\n    msg.temperature_C = parseInt(msg.response.val[5])\n} else if (msg.response.command === '0x00'){\n    if (msg.response.val[1] === \"0x01\"){\n        msg.power = \"ON\"\n    } else if (msg.response.val[1] === \"0x00\"){\n        msg.power = \"OFF\"\n    }\n    msg.volume = parseInt(msg.response.val[2])\n    if (msg.response.val[3] === \"0x01\"){\n        msg.mute = \"ON\"\n    } else if (msg.response.val[3] === \"0x00\"){\n        msg.mute = \"OFF\"\n    }\n    if (msg.response.val[4] === \"0x21\"){\n        msg.in = 'HDMI1'\n    } else if (msg.response.val[4] === \"0x23\"){\n        msg.in = 'HDMI2'\n    }\n    switch(msg.response.val[5]){\n        case '0x10':\n            msg.aspect = 'PC Mode 16:9';\n            break;\n        case '0x18':\n            msg.aspect = 'PC Mode 4:3';\n            break;\n        case '0x20':\n            msg.aspect = 'PC Mode Original Ratio';\n            break;\n        case '0x21':\n            msg.aspect = 'PC Mode 21:9';\n            break;\n        case '0x00':\n            msg.aspect = 'Auto Wide';\n            break;\n        case '0x01':\n            msg.aspect = '16:9';\n            break;\n        case '0x04':\n            msg.aspect = 'Zoom';\n            break;\n        case '0x05':\n            msg.aspect = 'Zoom 1';\n            break;\n        case '0x06':\n            msg.aspect = 'Zoom 2';\n            break;\n        case '0x09':\n            msg.aspect = 'Screen Fit';\n            break;\n        case '0x0B':\n            msg.aspect = '4:3';\n            break;\n        case '0x0C':\n            msg.aspect = 'Wide Fit';\n            break;\n        case '0x0D':\n            msg.aspect = 'Custom';\n            break;\n        case '0x0E':\n            msg.aspect = 'Smart View 1';\n            break;\n        case '0x0F':\n            msg.aspect = 'Smart View 2';\n            break;\n        case '0x31':\n            msg.aspect = 'Wide Zoom';\n            break;\n        case '0x32':\n            msg.aspect = '21:9';\n            break;\n    }\n}\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "x": 1030,
        "y": 880,
        "wires": [
            [
                "e20b5749.16ce88"
            ]
        ]
    }
]`
