# api documentation for  [json-schema (v0.2.3)](https://github.com/kriszyp/json-schema#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-json-schema.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-json-schema) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-json-schema.svg)](https://travis-ci.org/npmdoc/node-npmdoc-json-schema)
#### JSON Schema validation and specifications

[![NPM](https://nodei.co/npm/json-schema.png?downloads=true)](https://www.npmjs.com/package/json-schema)

[![apidoc](https://npmdoc.github.io/node-npmdoc-json-schema/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-json-schema_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-json-schema/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-json-schema/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-json-schema/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Kris Zyp"
    },
    "bugs": {
        "url": "https://github.com/kriszyp/json-schema/issues"
    },
    "dependencies": {},
    "description": "JSON Schema validation and specifications",
    "devDependencies": {
        "vows": "*"
    },
    "directories": {
        "lib": "./lib"
    },
    "dist": {
        "shasum": "b480c892e59a2f05954ce727bd3f2a4e882f9e13",
        "tarball": "https://registry.npmjs.org/json-schema/-/json-schema-0.2.3.tgz"
    },
    "gitHead": "07ae2c618b5f581dbc108e065f4f95dcf0a1d85f",
    "homepage": "https://github.com/kriszyp/json-schema#readme",
    "keywords": [
        "json",
        "schema"
    ],
    "licenses": [
        {
            "type": "AFLv2.1",
            "url": "http://trac.dojotoolkit.org/browser/dojo/trunk/LICENSE#L43"
        },
        {
            "type": "BSD",
            "url": "http://trac.dojotoolkit.org/browser/dojo/trunk/LICENSE#L13"
        }
    ],
    "main": "./lib/validate.js",
    "maintainers": [
        {
            "name": "kriszyp",
            "email": "kriszyp@gmail.com"
        }
    ],
    "name": "json-schema",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/kriszyp/json-schema.git"
    },
    "scripts": {
        "test": "echo TESTS DISABLED vows --spec test/*.js"
    },
    "version": "0.2.3"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module json-schema](#apidoc.module.json-schema)
1.  [function <span class="apidocSignatureSpan">json-schema.</span>_validate (instance, schema, options)](#apidoc.element.json-schema._validate)
1.  [function <span class="apidocSignatureSpan">json-schema.</span>checkPropertyChange (value, schema, property)](#apidoc.element.json-schema.checkPropertyChange)
1.  [function <span class="apidocSignatureSpan">json-schema.</span>mustBeValid (result)](#apidoc.element.json-schema.mustBeValid)
1.  [function <span class="apidocSignatureSpan">json-schema.</span>validate (instance, schema)](#apidoc.element.json-schema.validate)
1.  object <span class="apidocSignatureSpan">json-schema.</span>Integer
1.  object <span class="apidocSignatureSpan">json-schema.</span>links

#### [module json-schema.links](#apidoc.module.json-schema.links)
1.  boolean <span class="apidocSignatureSpan">json-schema.links.</span>cacheLinks
1.  [function <span class="apidocSignatureSpan">json-schema.links.</span>getLink (relation, instance, schema)](#apidoc.element.json-schema.links.getLink)
1.  [function <span class="apidocSignatureSpan">json-schema.links.</span>substitute (linkTemplate, instance)](#apidoc.element.json-schema.links.substitute)



# <a name="apidoc.module.json-schema"></a>[module json-schema](#apidoc.module.json-schema)

#### <a name="apidoc.element.json-schema._validate"></a>[function <span class="apidocSignatureSpan">json-schema.</span>_validate (instance, schema, options)](#apidoc.element.json-schema._validate)
- description and source-code
```javascript
_validate = function (instance, schema, options) {

	if (!options) options = {};
	var _changing = options.changing;

	function getType(schema){
		return schema.type || (primitiveConstructors[schema.name] == schema && schema.name.toLowerCase());
	}
	var errors = [];
	// validate a value against a property definition
	function checkProp(value, schema, path,i){

		var l;
		path += path ? typeof i == 'number' ? '[' + i + ']' : typeof i == 'undefined' ? '' : '.' + i : i;
		function addError(message){
			errors.push({property:path,message:message});
		}

		if((typeof schema != 'object' || schema instanceof Array) && (path || typeof schema != 'function') && !(schema && getType(schema
))){
			if(typeof schema == 'function'){
				if(!(value instanceof schema)){
					addError("is not an instance of the class/constructor " + schema.name);
				}
			}else if(schema){
				addError("Invalid schema/property definition " + schema);
			}
			return null;
		}
		if(_changing && schema.readonly){
			addError("is a readonly field, it can not be changed");
		}
		if(schema['extends']){ // if it extends another schema, it must pass that schema as well
			checkProp(value,schema['extends'],path,i);
		}
		// validate a value against a type definition
		function checkType(type,value){
			if(type){
				if(typeof type == 'string' && type != 'any' &&
						(type == 'null' ? value !== null : typeof value != type) &&
						!(value instanceof Array && type == 'array') &&
						!(value instanceof Date && type == 'date') &&
						!(type == 'integer' && value%1===0)){
					return [{property:path,message:(typeof value) + " value found, but a " + type + " is required"}];
				}
				if(type instanceof Array){
					var unionErrors=[];
					for(var j = 0; j < type.length; j++){ // a union type
						if(!(unionErrors=checkType(type[j],value)).length){
							break;
						}
					}
					if(unionErrors.length){
						return unionErrors;
					}
				}else if(typeof type == 'object'){
					var priorErrors = errors;
					errors = [];
					checkProp(value,type,path);
					var theseErrors = errors;
					errors = priorErrors;
					return theseErrors;
				}
			}
			return [];
		}
		if(value === undefined){
			if(schema.required){
				addError("is missing and it is required");
			}
		}else{
			errors = errors.concat(checkType(getType(schema),value));
			if(schema.disallow && !checkType(schema.disallow,value).length){
				addError(" disallowed value was matched");
			}
			if(value !== null){
				if(value instanceof Array){
					if(schema.items){
						var itemsIsArray = schema.items instanceof Array;
						var propDef = schema.items;
						for (i = 0, l = value.length; i < l; i += 1) {
							if (itemsIsArray)
								propDef = schema.items[i];
							if (options.coerce)
								value[i] = options.coerce(value[i], propDef);
							errors.concat(checkProp(value[i],propDef,path,i));
						}
					}
					if(schema.minItems && value.length < schema.minItems){
						addError("There must be a minimum of " + schema.minItems + " in the array");
					}
					if(schema.maxItems && value.length > schema.maxItems){
						addError("There must be a maximum of " + schema.maxItems + " in the array");
					}
				}else if(schema.properties || schema.additionalProperties){
					errors.concat(checkObj(value, schema.properties, path, schema.additionalProperties));
				}
				if(schema.pattern && typeof value == 'string' && !value.match(schema.pattern)){
					addError("does not match the regex pattern " + schema.pattern);
				}
				if(schema.maxLength && typeof value == 'string' && value.length > schema.maxLength){
					addError("may only be " + schema.maxLength + " characters long");
				}
				if(schema.minLength && typeof value == 'string' && value.length < schema.minLength){
					addError("must be at least " + schema.minLength + " characters long");
				}
				if(typeof schema.minimum !== undefined && typeof value == typeof schema.minimum &&
						schema.minimum > value){
					addErro ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.json-schema.checkPropertyChange"></a>[function <span class="apidocSignatureSpan">json-schema.</span>checkPropertyChange (value, schema, property)](#apidoc.element.json-schema.checkPropertyChange)
- description and source-code
```javascript
checkPropertyChange = function (value, schema, property) {
		// Summary:
		// 		The checkPropertyChange method will check to see if an value can legally be in property with the given schema
		// 		This is slightly different than the validate method in that it will fail if the schema is readonly and it will
		// 		not check for self-validation, it is assumed that the passed in value is already internally valid.
		// 		The checkPropertyChange method will return the same object type as validate, see JSONSchema.validate for
		// 		information.
		//
		return validate(value, schema, {changing: property || "property"});
	}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.json-schema.mustBeValid"></a>[function <span class="apidocSignatureSpan">json-schema.</span>mustBeValid (result)](#apidoc.element.json-schema.mustBeValid)
- description and source-code
```javascript
mustBeValid = function (result){
	//	summary:
	//		This checks to ensure that the result is valid and will throw an appropriate error message if it is not
	// result: the result returned from checkPropertyChange or validate
	if(!result.valid){
		throw new TypeError(result.errors.map(function(error){return "for property " + error.property + ': ' + error.message;}).join(",\n"));
	}
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.json-schema.validate"></a>[function <span class="apidocSignatureSpan">json-schema.</span>validate (instance, schema)](#apidoc.element.json-schema.validate)
- description and source-code
```javascript
function validate(instance, schema) {
		// Summary:
		//  	To use the validator call JSONSchema.validate with an instance object and an optional schema object.
		// 		If a schema is provided, it will be used to validate. If the instance object refers to a schema (self-validating),
		// 		that schema will be used to validate and the schema parameter is not necessary (if both exist,
		// 		both validations will occur).
		// 		The validate method will return an object with two properties:
		// 			valid: A boolean indicating if the instance is valid by the schema
		// 			errors: An array of validation errors. If there are no errors, then an
		// 					empty list will be returned. A validation error will have two properties:
		// 						property: which indicates which property had the error
		// 						message: which indicates what the error was
		//
		return validate(instance, schema, {changing: false});//, coerce: false, existingOnly: false});
	}
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.json-schema.links"></a>[module json-schema.links](#apidoc.module.json-schema.links)

#### <a name="apidoc.element.json-schema.links.getLink"></a>[function <span class="apidocSignatureSpan">json-schema.links.</span>getLink (relation, instance, schema)](#apidoc.element.json-schema.links.getLink)
- description and source-code
```javascript
getLink = function (relation, instance, schema){
	// gets the URI of the link for the given relation based on the instance and schema
	// for example:
	// getLink(
	// 		"brother",
	// 		{"brother_id":33},
	// 		{links:[{rel:"brother", href:"Brother/{brother_id}"}]}) ->
	//	"Brother/33"
	var links = schema.__linkTemplates;
	if(!links){
		links = {};
		var schemaLinks = schema.links;
		if(schemaLinks && schemaLinks instanceof Array){
			schemaLinks.forEach(function(link){
	/*			// TODO: allow for multiple same-name relations
				if(links[link.rel]){
					if(!(links[link.rel] instanceof Array)){
						links[link.rel] = [links[link.rel]];
					}
				}*/
				links[link.rel] = link.href;
			});
		}
		if(exports.cacheLinks){
			schema.__linkTemplates = links;
		}
	}
	var linkTemplate = links[relation];
	return linkTemplate && exports.substitute(linkTemplate, instance);
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.json-schema.links.substitute"></a>[function <span class="apidocSignatureSpan">json-schema.links.</span>substitute (linkTemplate, instance)](#apidoc.element.json-schema.links.substitute)
- description and source-code
```javascript
substitute = function (linkTemplate, instance){
	return linkTemplate.replace(/\{([^\}]*)\}/g, function(t, property){
			var value = instance[decodeURIComponent(property)];
			if(value instanceof Array){
				// the value is an array, it should produce a URI like /Table/(4,5,8) and store.get() should handle that as an array of values
				return '(' + value.join(',') + ')';
			}
			return value;
		});
}
```
- example usage
```shell
...
			});
		}
		if(exports.cacheLinks){
			schema.__linkTemplates = links;
		}
	}
	var linkTemplate = links[relation];
	return linkTemplate && exports.substitute(linkTemplate, instance);
};

exports.substitute = function(linkTemplate, instance){
	return linkTemplate.replace(/\{([^\}]*)\}/g, function(t, property){
			var value = instance[decodeURIComponent(property)];
			if(value instanceof Array){
				// the value is an array, it should produce a URI like /Table/(4,5,8) and store.get() should handle that as an array of values
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
