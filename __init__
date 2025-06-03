"""Component Lite Chain 01 module"""

from mgear.pymaya import datatypes

from mgear.shifter import component

from mgear.core import attribute

from mgear.core import transform, primitive, vector

##########################################################
# COMPONENT
##########################################################


class Component(component.Main):
    """Shifter component Class"""

    # =====================================================
    # OBJECTS
    # =====================================================
    def addObjects(self):
        """Add all the objects needed to create the component."""
        self.tr_params = ["tx", "ty", "tz", "rx", "ry", "rz"]
        # self.normal = self.guide.blades["blade"].z * -1
        # self.binormal = self.guide.blades["blade"].x

        self.WIP = self.options["mode"]
        if self.negate and self.settings["overrideNegate"]:
            self.negate = False
            self.n_factor = 1

        if self.settings["overrideNegate"]:
            self.mirror_conf = [0, 0, 1,
                                1, 1, 0,
                                0, 0, 0]
        else:
            self.mirror_conf = [0, 0, 0,
                                0, 0, 0,
                                0, 0, 0]

        # FK controllers ------------------------------------
        self.fk_npo = []
        self.fk_ctl = []
        t = self.guide.tra["root"]

        parent = self.root
        tOld = False
        fk_ctl = None
        self.previusTag = self.parentCtlTag

        # transforms = []
        # for i in range(len(self.guide.atra) - 1):
        #     transforms.append(transform.getTransform(self.guide.atra[i]))
        for i, t in enumerate(self.guide.atra[:-1]):
            # dist = vector.getDistance(self.guide.apos[i],
            #                           self.guide.apos[i + 1])
            t = transform.setMatrixScale(t)
            if self.settings["neutralpose"] or not tOld:
                tnpo = t
            else:
                tnpo = transform.setMatrixPosition(
                    tOld,
                    transform.getPositionFromMatrix(t))

            fk_npo = primitive.addTransform(
                parent, self.getName("fk%s_npo" % i), tnpo)
            fk_ctl = self.addCtl(
                fk_npo,
                "fk%s_ctl" % i,
                t,
                self.color_fk,
                "circle",
                w=self.size * .4,
                ro=datatypes.Vector([0, 0, 1.5708]),
                tp=self.previusTag,
                mirrorConf=self.mirror_conf)

            self.fk_npo.append(fk_npo)
            self.fk_ctl.append(fk_ctl)
            tOld = t
            self.previusTag = fk_ctl
            parent = fk_ctl


        # ================================================
        # IK-style extra controls for hip and chest
        # ================================================

        t = transform.getTransform(self.fk_ctl[0])  # ensure clean local transform
        self.ik0_npo = primitive.addTransform(
            self.fk_ctl[0], self.getName("ik0_npo"), t)
        self.ik0_ctl = self.addCtl(self.ik0_npo,
                                   "ik0_ctl",
                                   t,
                                   self.color_ik,
                                   "compas",
                                   w=self.size,
                                   tp=self.parentCtlTag)

        #attribute.setKeyableAttributes(self.ik0_ctl, self.tr_params)
        #attribute.setRotOrder(self.ik0_ctl, "ZXY")
        #attribute.setInvertMirror(self.ik0_ctl, ["tx", "ry", "rz"])

        
        t = transform.setMatrixPosition(t, self.guide.apos[-2])
        self.ik1_npo = primitive.addTransform(
            self.fk_ctl[-1], self.getName("ik1_npo"), t)

        self.ik1_ctl = self.addCtl(self.ik1_npo,
                                   "ik1_ctl",
                                   t,
                                   self.color_ik,
                                   "compas",
                                   w=self.size,
                                   tp=self.ik0_ctl)

        #attribute.setKeyableAttributes(self.ik1_ctl, self.tr_params)
        #attribute.setRotOrder(self.ik1_ctl, "ZXY")
        #attribute.setInvertMirror(self.ik1_ctl, ["tx", "ry", "rz"])

    # =====================================================
    # ADD JOINTS
    # =====================================================

        for i, fk_ctl in enumerate(self.fk_ctl):
            if i == 0:
                self.jnt_pos.append([self.ik0_ctl, i, None, False])  
            elif i == len(self.fk_ctl) - 1:
                self.jnt_pos.append([self.ik1_ctl, i, None, False])  
            else:
                self.jnt_pos.append([fk_ctl, i, None, False])  

            print("Joint Drivers:", [j[0] for j in self.jnt_pos])

    # =====================================================
    # ATTRIBUTES
    # =====================================================
    def addAttributes(self):
        """Create the anim and setupr rig attributes for the component"""

        return

    # =====================================================
    # OPERATORS
    # =====================================================
    def addOperators(self):
        """Create operators and set the relations for the component rig

        Apply operators, constraints, expressions to the hierarchy.
        In order to keep the code clean and easier to debug,
        we shouldn't create any new object in this method.

        """
        return

    # =====================================================
    # CONNECTOR
    # =====================================================
    def setRelation(self):
        """Set the relation beetween object from guide to rig"""

        self.relatives["root"] = self.fk_ctl[0]
        self.controlRelatives["root"] = self.fk_ctl[0]
        self.jointRelatives["root"] = 0
        for i in range(0, len(self.fk_ctl) - 1):
            self.relatives["%s_loc" % i] = self.fk_ctl[i + 1]
            self.controlRelatives["%s_loc" % i] = self.fk_ctl[i + 1]
            self.jointRelatives["%s_loc" % i] = i + 1
            self.aliasRelatives["%s_ctl" % i] = i + 1
        self.relatives["%s_loc" % (len(self.fk_ctl) - 1)] = self.fk_ctl[-1]
        self.controlRelatives["%s_loc" % (
            len(self.fk_ctl) - 1)] = self.fk_ctl[-1]
        self.jointRelatives["%s_loc" % (
            len(self.fk_ctl) - 1)] = len(self.fk_ctl) - 1
        self.aliasRelatives["%s_loc" % (
            len(self.fk_ctl) - 1)] = len(self.fk_ctl) - 1

# Error: NameError: file C:\Users/damer/Documents/maya/modules/scripts/mgear/shifter_classic_components\simple_spine\__init__.py line 125: name 'self' is not defined #
