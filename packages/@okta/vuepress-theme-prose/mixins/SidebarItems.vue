<script>
import _ from "lodash";
import { getGuidesInfo, guideFromPath } from "../util/guides";
import {
  concepts,
  guides,
  languagesSdk,
  reference,
  releaseNotes
} from "../const/navbar.const";
      
export default {
  data() {
    return {
      navigation: [],
      link_id: 0,
    };
  },
  methods: {
    getNavigation() {
      const homeLink = { title: "Home", path: "/" };
      return [
        homeLink,
        ...this.getGuides(),
        ..._.cloneDeep(concepts),
        ..._.cloneDeep(reference),
        ..._.cloneDeep(languagesSdk),
        ..._.cloneDeep(releaseNotes)
      ];
    },
    getNavigationData() {
      this.navigation = this.getNavigation().map(nav => {
        this.addStatesToLink(nav);     
        return nav;
      });
      return this.navigation
    },
    addStatesToLink(link) {
      // Reset iHaveChildrenActive value.
      link.iHaveChildrenActive = false;

      if (link.path) {
        // Add state to leaf link
        link.iHaveChildrenActive = link.path === this.$page.regularPath;
      }
      if (link.subLinks) {
        for (const subLink of link.subLinks) {
          // if link has active children - continue with the rest of its sublinks
          if (link.iHaveChildrenActive) {
            this.addStatesToLink(subLink);
          } else {
            link.iHaveChildrenActive = this.addStatesToLink(subLink);
          }
        }
      }

      return link.iHaveChildrenActive;
    },
    getGuides() {
      const pages = this.$site.pages;
      const guidesInfo = getGuidesInfo({ pages });
      let navs = _.cloneDeep(guides);
      const framework = guideFromPath(this.$route.path).framework;
      navs.forEach(nav => {
        let queue = new Array();
        queue.push(nav);
        let current = queue.pop();
        while (current) {
          if (current?.subLinks) {
            queue.push(...current.subLinks);
          } else if (current?.guideName) {
            // add sections
            current.subLinks = [];
            const guide = guidesInfo.byName[current.guideName];

            if (Array.isArray(guide?.sections)) {
              const [firstSection] = guide.sections;

              // Special value for guide that only has one section and should be
              // linked at the parent
              if (guide.sections.length === 1 && firstSection.name === 'main') {
                current.title = firstSection.title;
                current.path = firstSection.makeLink(guide.frameworks.includes(framework) ? framework : guide.mainFramework);
                current.frameworks = guide.frameworks;
              } else {
                guide.sections.forEach(section => {
                  current.subLinks.push({
                    title: section.title,
                    path: section.makeLink(
                      guide.frameworks.includes(framework)
                        ? framework
                        : guide.mainFramework
                    ),
                    frameworks: guide.frameworks
                  });
                });
              }
            }
          }
          current = queue.pop();
        }

      });
      return navs;
    }
  }
}
</script>