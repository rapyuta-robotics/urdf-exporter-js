# urdf-exporter-js

[![npm version](https://img.shields.io/npm/v/urdf-exporter.svg?style=flat-square)](https://www.npmjs.com/package/urdf-exporter)
[![travis build](https://img.shields.io/travis/gkjohnson/urdf-exporter-js.svg?style=flat-square)](https://travis-ci.org/gkjohnson/urdf-exporter-js)
[![lgtm code quality](https://img.shields.io/lgtm/grade/javascript/g/gkjohnson/urdf-exporter-js.svg?style=flat-square&label=code-quality)](https://lgtm.com/projects/g/gkjohnson/urdf-exporter-js/)

Utility for exporting THREE.js object trees as a URDF file.

## URDFExporter
```js
// Function callback for defining the attributes for
// the joint to be generated for the associated link.
// If not provided, `fixed` joint type is used.
function jointFunc (obj, childName, parentName) {
    if (obj.urdf) {
        return {
            name: obj.name,
            type: obj.jointType,
            axis: obj.axis || new THREE.Vector3(0, 0, 1),
            limit: {
                lower: obj.limit.lower,
                upper: obj.limit.upper,
            },
            isLeaf: false
        };
    }
    return {};
}

// Generate the URDF file and associate meshes and textures
const exporter = new URDFExporter();
const res = exporter.parse(robot, jointFunc, { collapse: true, robotName: 'T12' });

// Save out a zip using JSZip (https://stuk.github.io/jszip/)
// with the file structure

// /<name>.urdf
// /meshes/<mesh>.<ext>
// /textures/<texture>.<ext>
const zip = new JSZip();
zip.file('T12.URDF', res.data);
res.meshes.forEach(m => zip.file(`${ m.directory }${ m.name }.${ m.ext }`, m.data));
res.textures.forEach(t => zip.file($ {t.directory }`${ t.name }.${ t.ext }`, m.data));

zip
    .generateAsync({ type: 'uint8array' })
    .then(zipdata => saveData(zipdata, 't12urdf.zip'));
```

### URDFExporter.parse(object, jointfunc, options = {}) : Object

Processes the given `object` into a URDF file and assets. Returns an object of the form
```js
{
  data: <string>,
  meshes: [{ directory, name, ext, data }, ...],
  textures: [{ directory, name, ext, data }, ...]
}
```

#### object : THREE.Object3D
_required_

The THREE.js OBject3D to convert to a URDF file and associated meshes and textures.

#### jointfunc(object, childLinkName, parentLinkName) : Function
_required_

A function used to define the attributes for a joint connecting a link to its parent link. Takes the object being converted to a link, the child link name and parent link name. An example with the full set of output data:

```js
function joinfunc(childObject, childLinkName, parentLinkName) {
  return {

    // The name of the joint.
    // Defaults to '_joint_<number>'.
    name: `${parentLinkName}2${childLinkName}`,

    // The URDF joint articulation type.
    // Defaults to 'fixed'.
    type: `revolute`,

    // The axis of movement or rotation for any given joint.
    // Required for non-fixed joints.
    axis: new THREE.Vector3(1, 0, 0),

    // Descriptions of the limits for any given joint
    limits: {
      lower: -30,  // defaults to 0
      upper: 30,   // defaults to 0
      velocity: 1, // defaults to 0
      effort: 1,   // defaults to 0
    },

    // A field indicating that the link for the provided childObject
    // should be considered a leaf link and no children should be
    // added for it.
    // Defaults to false.
    isLeaf: false,
  }
}
```

#### options
##### createMeshCb(obj, linkname, meshFormat) : Function

A function used to generate mesh data for a given node. STL or DAE files are generated by default. An example function with the full set of output data:
```js
function createMesh(obj, linkName, meshFormat) {
  // meshFormat is passed from `options.meshFormat`.
  // Can be ignored.

  const res = this.ColladaExporter.parse(o);
  return {
      // File name
      name: linkName,

      // File extension
      ext: 'dae',

      // File content
      data: res.data,

      // Array of texture data to save out. Object must
      // include name, ext, and data to write out.
      textures: res.textures,

      // The material information to include. Material
      // is excluded if this is not defined.
      // This is not needed for mesh files that include
      // material information.
      material: {

          color: obj.material.color,
          texture: obj.material.map

      }
  }
}
```

##### pathPrefix : String

The prefix to prepend to all file paths for meshes and textures. Defaults to `./`.

This could be set to a ROS package URI, such as `package://robot-name/`.

##### collapse : Boolean

Attempts to collapse unnecessary joints to remove any meaningless links in the URDF (identity transform, fixed). Experimental. Defaults to false.

##### meshFormat : String

The preferred format to export meshes as. Is passed to the `meshFunc` optional function. Defaults to `dae`.

##### robotname : String

The name of the robot to add into the `<robot>` tag `name` field. Defaults to the name of the object being converted.

